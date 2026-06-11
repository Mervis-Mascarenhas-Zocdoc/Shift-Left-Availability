# Interop Availability API - API Test Mapping

## tests/ApiTests/ExclusionsApiTests.cs

### BatchAdd Exclusions

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 1 | | BatchAddExclusions_Authorization (PatientUser_Forbidden) | Authorization check: Patient users cannot batch-add exclusions. | Get patient JWT headers -> POST /v1/exclusions with a single exclusion payload -> Assert HTTP 403 Forbidden. | Verifies that patient-authenticated users are forbidden from creating exclusions. | BatchAdd authorization. In scope: patient user forbidden. Out of scope: other auth scenarios; exclusion content validation. |
| 2 | | BatchAddExclusions_Authorization (ProviderUser_CorrectPracticeId_Ok) | Authorization check: Provider user with matching practiceId can batch-add exclusions for all entity types (ProviderId, ProviderLocationId, TemplateId). | Get provider JWT headers with correct practiceId -> POST /v1/exclusions with exclusions for all entity types -> Assert HTTP 200 OK. | Verifies provider users with correct practice are authorized to add exclusions. | BatchAdd authorization. In scope: provider user correct practice OK. Out of scope: exclusion content validation. |
| 3 | | BatchAddExclusions_Authorization (ProviderUser_IncorrectPracticeId_Forbidden) | Authorization check: Provider user with wrong practiceId is forbidden. | Get provider JWT headers with incorrect practiceId -> POST /v1/exclusions -> Assert HTTP 403 Forbidden. | Verifies provider users with mismatched practice are forbidden. | BatchAdd authorization. In scope: provider user wrong practice forbidden. Out of scope: correct practice scenarios. |
| 4 | | BatchAddExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndRole_Ok) | Authorization check: Provider RBAC user with correct rbacPracticeId and PracticeFullAdmin role can add exclusions. | Get practice RBAC JWT with correct rbacPracticeId + PracticeFullAdmin role -> POST /v1/exclusions -> Assert HTTP 200 OK. | Verifies RBAC provider with correct practice and admin role is authorized. | BatchAdd authorization. In scope: RBAC correct practice + role OK. Out of scope: incorrect role/practice combos. |
| 5 | | BatchAddExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndIncorrectRole_Forbidden) | Authorization check: Provider RBAC user with correct rbacPracticeId but PracticeBilling role (not admin) is forbidden. | Get practice RBAC JWT with correct rbacPracticeId + PracticeBilling role -> POST /v1/exclusions -> Assert HTTP 403 Forbidden. | Verifies RBAC provider with wrong role is forbidden even with correct practice. | BatchAdd authorization. In scope: RBAC correct practice + wrong role forbidden. Out of scope: correct role scenarios. |
| 6 | | BatchAddExclusions_Authorization (ProviderUser_IncorrectRbacPracticeIdAndCorrectRole_Forbidden) | Authorization check: Provider RBAC user with wrong rbacPracticeId but correct role is forbidden. | Get practice RBAC JWT with wrong rbacPracticeId + PracticeFullAdmin role -> POST /v1/exclusions -> Assert HTTP 403 Forbidden. | Verifies RBAC provider with wrong practice is forbidden even with correct role. | BatchAdd authorization. In scope: RBAC wrong practice + correct role forbidden. Out of scope: correct practice scenarios. |
| 7 | | BatchAddExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndPracticeAppointmentManagementRole_Ok) | Authorization check: Provider RBAC user with PracticeAppointmentManagement role can add exclusions. | Get practice RBAC JWT with correct rbacPracticeId + PracticeAppointmentManagement role -> POST /v1/exclusions -> Assert HTTP 200 OK. | Verifies RBAC provider with appointment management role is authorized. | BatchAdd authorization. In scope: RBAC appointment management role OK. Out of scope: other roles. |
| 8 | | BatchAddExclusions_Authorization (CsrUser_WithoutReadWriteRole_Forbidden) | Authorization check: CSR user without ReadWrite role is forbidden. | Get CSR JWT without ReadWrite role -> POST /v1/exclusions -> Assert HTTP 403 Forbidden. | Verifies CSR users without exclusions ReadWrite role cannot add exclusions. | BatchAdd authorization. In scope: CSR without role forbidden. Out of scope: CSR with role. |
| 9 | | BatchAddExclusions_Authorization (CsrUser_WithReadWriteRole_Ok) | Authorization check: CSR user with ReadWrite role can add exclusions. | Get CSR JWT with CsrCanModifyTimesInProviderCalendar role -> POST /v1/exclusions -> Assert HTTP 200 OK. | Verifies CSR users with the correct role are authorized. | BatchAdd authorization. In scope: CSR with role OK. Out of scope: CSR without role. |
| 10 | | BatchAddExclusions_Authorization (ServiceAccount_Ok) | Authorization check: Service account can add exclusions. | Get service account JWT (InteropAvailabilityApiExclusionsReadWrite) -> POST /v1/exclusions -> Assert HTTP 200 OK. | Verifies service accounts are authorized to add exclusions. | BatchAdd authorization. In scope: service account OK. Out of scope: other auth types. |
| 11 | | BatchAddExclusions_Authorization (MissingJwt_Unauthorized) | Authorization check: Missing JWT returns 401. | POST /v1/exclusions with no auth headers -> Assert HTTP 401 Unauthorized. | Verifies unauthenticated requests are rejected. | BatchAdd authorization. In scope: missing JWT unauthorized. Out of scope: valid JWT scenarios. |
| 12 | | BatchAddExclusions_Success | Happy path: Create a single exclusion, verify all returned fields, then confirm via GetByIds and GetByRangeV2. | Get service account headers -> Create AddExclusionParams (Busy type) -> POST /v1/exclusions -> Assert 1 exclusion returned -> Verify Id starts with "ae_" -> Verify EntityId, StartDateTimeUtc, EndDateTimeUtc, ExclusionSource, ExclusionType, AppointmentId match input -> GET by ID confirms 1 exclusion -> GET by range V2 confirms exclusion exists in range. | End-to-end creation and read-back of a single exclusion with all field validation. | BatchAdd success flow. In scope: create; field validation; read-back via GetByIds and GetByRangeV2. Out of scope: update; delete; authorization. |
| 13 | | BatchAddExclusions_GivenNoExclusions_BadRequest | Validation: Sending empty exclusions list returns 400. | Get service account headers -> POST /v1/exclusions with empty list -> Assert HTTP 400 BadRequest. | Verifies the API rejects empty exclusion batches. | BatchAdd input validation. In scope: empty list bad request. Out of scope: valid input; too many items. |
| 14 | | BatchAddExclusions_GivenTooManyExclusions_BadRequest | Validation: Sending 105 exclusions (over limit) returns 400. | Get service account headers -> POST /v1/exclusions with 105 identical exclusions -> Assert HTTP 400 BadRequest. | Verifies the API enforces a maximum batch size for exclusion creation. | BatchAdd input validation. In scope: over-limit bad request. Out of scope: valid counts; empty list. |
| 15 | | BatchAddExclusions_GivenAppointmentIdForNonAppointmentType_BadRequest | Validation: Setting AppointmentId on a Busy-type exclusion returns 400. | Get service account headers -> Create Busy exclusion with AppointmentId set -> POST /v1/exclusions -> Assert HTTP 400 BadRequest. | Verifies AppointmentId is only allowed for Appointment-type exclusions. | BatchAdd input validation. In scope: type/appointmentId mismatch. Out of scope: valid Appointment type with AppointmentId. |

### BatchGetByIds

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 16 | | BatchGetExclusionsByIds_Authorization (PatientUser_Forbidden) | Authorization: Patient users cannot get exclusions by IDs. | Get patient JWT -> POST /v1/exclusions/id~batchGet -> Assert HTTP 403 Forbidden. | Patient users are forbidden from reading exclusions by ID. | GetByIds authorization. In scope: patient forbidden. Out of scope: other auth. |
| 17 | | BatchGetExclusionsByIds_Authorization (ProviderUser_CorrectPracticeId_Ok) | Authorization: Provider user with correct practiceId can get exclusions by IDs. | Get provider JWT with correct practiceId -> Add exclusions for all entities -> POST /v1/exclusions/id~batchGet -> Assert HTTP 200 OK. | Provider with correct practice is authorized. | GetByIds authorization. In scope: provider correct practice OK. |
| 18 | | BatchGetExclusionsByIds_Authorization (ProviderUser_IncorrectPracticeId_Forbidden) | Authorization: Provider user with wrong practiceId is forbidden. | Get provider JWT with wrong practiceId -> POST /v1/exclusions/id~batchGet -> Assert HTTP 403 Forbidden. | Provider with mismatched practice is forbidden. | GetByIds authorization. In scope: provider wrong practice forbidden. |
| 19 | | BatchGetExclusionsByIds_Authorization (ProviderUser_CorrectRbacPracticeIdAndRole_Ok) | Authorization: RBAC provider with correct practice and PracticeFullAdmin role can get exclusions. | Get RBAC JWT with correct rbacPracticeId + PracticeFullAdmin -> POST /v1/exclusions/id~batchGet -> Assert HTTP 200 OK. | RBAC provider with correct practice and role is authorized. | GetByIds authorization. In scope: RBAC correct practice + role OK. |
| 20 | | BatchGetExclusionsByIds_Authorization (ProviderUser_CorrectRbacPracticeIdAndIncorrectRole_Forbidden) | Authorization: RBAC provider with correct practice but wrong role (PracticeBilling) is forbidden. | Get RBAC JWT with correct rbacPracticeId + PracticeBilling -> POST /v1/exclusions/id~batchGet -> Assert HTTP 403 Forbidden. | RBAC provider with wrong role is forbidden. | GetByIds authorization. In scope: RBAC correct practice + wrong role forbidden. |
| 21 | | BatchGetExclusionsByIds_Authorization (ProviderUser_IncorrectRbacPracticeIdAndCorrectRole_Forbidden) | Authorization: RBAC provider with wrong practice but correct role is forbidden. | Get RBAC JWT with wrong rbacPracticeId + PracticeFullAdmin -> POST /v1/exclusions/id~batchGet -> Assert HTTP 403 Forbidden. | RBAC provider with wrong practice is forbidden even with correct role. | GetByIds authorization. In scope: RBAC wrong practice + correct role forbidden. |
| 22 | | BatchGetExclusionsByIds_Authorization (ProviderUser_CorrectRbacPracticeIdAndPracticeAppointmentManagementRole_Ok) | Authorization: RBAC provider with PracticeAppointmentManagement role can get exclusions. | Get RBAC JWT with correct rbacPracticeId + PracticeAppointmentManagement -> POST /v1/exclusions/id~batchGet -> Assert HTTP 200 OK. | RBAC provider with appointment management role is authorized. | GetByIds authorization. In scope: RBAC appointment management role OK. |
| 23 | | BatchGetExclusionsByIds_Authorization (CsrUser_WithoutReadWriteRole_Ok) | Authorization: CSR user WITHOUT ReadWrite role CAN read exclusions (read allowed for all CSR). | Get CSR JWT without special role -> POST /v1/exclusions/id~batchGet -> Assert HTTP 200 OK. | CSR users can read exclusions by ID regardless of ReadWrite role. | GetByIds authorization. In scope: CSR without role still OK for reads. |
| 24 | | BatchGetExclusionsByIds_Authorization (CsrUser_WithReadWriteRole_Ok) | Authorization: CSR user with ReadWrite role can read exclusions. | Get CSR JWT with role -> POST /v1/exclusions/id~batchGet -> Assert HTTP 200 OK. | CSR users with role are authorized. | GetByIds authorization. In scope: CSR with role OK. |
| 25 | | BatchGetExclusionsByIds_Authorization (ServiceAccount_Ok) | Authorization: Service account can read exclusions. | Get service account JWT -> POST /v1/exclusions/id~batchGet -> Assert HTTP 200 OK. | Service accounts are authorized. | GetByIds authorization. In scope: service account OK. |
| 26 | | BatchGetExclusionsByIds_Authorization (MissingJwt_Unauthorized) | Authorization: Missing JWT returns 401. | POST /v1/exclusions/id~batchGet with no headers -> Assert HTTP 401 Unauthorized. | Unauthenticated requests are rejected. | GetByIds authorization. In scope: missing JWT unauthorized. |
| 27 | | BatchGetExclusionsByIds_GivenNoExclusionIds_BadRequest | Validation: Empty ID list returns 400. | Get service account headers -> POST /v1/exclusions/id~batchGet with empty list -> Assert HTTP 400 BadRequest. | API rejects empty ID batches. | GetByIds validation. In scope: empty list bad request. |
| 28 | | BatchGetExclusionsByIds_GivenTooManyExclusionIds_BadRequest | Validation: 505 IDs (over limit) returns 400. | Get service account headers -> POST /v1/exclusions/id~batchGet with 505 IDs -> Assert HTTP 400 BadRequest. | API enforces max batch size for ID lookups. | GetByIds validation. In scope: over-limit bad request. |
| 29 | | BatchGetExclusionsByIds_Success | Happy path: Create an exclusion, get it by ID (including a non-existing ID), verify only the real one is returned with correct fields. | Get service account headers -> Create Appointment exclusion -> GET by IDs [real_id, "ae_non_existing_id"] -> Assert 1 result -> Verify all fields match. | End-to-end create and read-back by ID, non-existing IDs are silently omitted. | GetByIds success. In scope: create; get by ID; field validation; non-existing ID handling. Out of scope: delete; update. |

### BatchGetByRangeV2

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 30 | | BatchGetExclusionsByRangeV2_Authorization (PatientUser_Forbidden) | Authorization: Patient users cannot get exclusions by range. | Get patient JWT -> POST /v2/exclusions/range~batchGet -> Assert HTTP 403 Forbidden. | Patient users are forbidden from reading exclusions by range. | GetByRangeV2 authorization. In scope: patient forbidden. |
| 31 | | BatchGetExclusionsByRangeV2_Authorization (ProviderUser_CorrectPracticeId_Ok) | Authorization: Provider with correct practice can query by range. | Get provider JWT correct practice -> POST /v2/exclusions/range~batchGet -> Assert HTTP 200 OK. | Provider with correct practice authorized. | GetByRangeV2 authorization. In scope: provider correct practice OK. |
| 32 | | BatchGetExclusionsByRangeV2_Authorization (ProviderUser_IncorrectPracticeId_Forbidden) | Authorization: Provider with wrong practice is forbidden. | Get provider JWT wrong practice -> POST /v2/exclusions/range~batchGet -> Assert HTTP 403 Forbidden. | Provider with wrong practice forbidden. | GetByRangeV2 authorization. In scope: provider wrong practice forbidden. |
| 33 | | BatchGetExclusionsByRangeV2_Authorization (ProviderUser_CorrectRbacPracticeIdAndRole_Ok) | Authorization: RBAC provider correct practice + PracticeFullAdmin OK. | Get RBAC JWT -> POST /v2/exclusions/range~batchGet -> Assert HTTP 200 OK. | RBAC authorized. | GetByRangeV2 authorization. In scope: RBAC correct practice + role OK. |
| 34 | | BatchGetExclusionsByRangeV2_Authorization (ProviderUser_CorrectRbacPracticeIdAndIncorrectRole_Forbidden) | Authorization: RBAC correct practice + PracticeBilling forbidden. | Get RBAC JWT wrong role -> POST /v2/exclusions/range~batchGet -> Assert HTTP 403 Forbidden. | RBAC wrong role forbidden. | GetByRangeV2 authorization. |
| 35 | | BatchGetExclusionsByRangeV2_Authorization (ProviderUser_IncorrectRbacPracticeIdAndCorrectRole_Forbidden) | Authorization: RBAC wrong practice + correct role forbidden. | Get RBAC JWT wrong practice -> POST /v2/exclusions/range~batchGet -> Assert HTTP 403 Forbidden. | RBAC wrong practice forbidden. | GetByRangeV2 authorization. |
| 36 | | BatchGetExclusionsByRangeV2_Authorization (ProviderUser_CorrectRbacPracticeIdAndPracticeAppointmentManagementRole_Ok) | Authorization: RBAC PracticeAppointmentManagement OK. | Get RBAC JWT -> POST /v2/exclusions/range~batchGet -> Assert HTTP 200 OK. | RBAC appointment management authorized. | GetByRangeV2 authorization. |
| 37 | | BatchGetExclusionsByRangeV2_Authorization (CsrUser_WithoutReadWriteRole_Ok) | Authorization: CSR without ReadWrite can still read by range. | Get CSR JWT -> POST /v2/exclusions/range~batchGet -> Assert HTTP 200 OK. | CSR read is allowed without write role. | GetByRangeV2 authorization. |
| 38 | | BatchGetExclusionsByRangeV2_Authorization (CsrUser_WithReadWriteRole_Ok) | Authorization: CSR with ReadWrite OK. | Get CSR JWT with role -> POST /v2/exclusions/range~batchGet -> Assert HTTP 200 OK. | CSR with role authorized. | GetByRangeV2 authorization. |
| 39 | | BatchGetExclusionsByRangeV2_Authorization (ServiceAccount_Ok) | Authorization: Service account OK. | Get service account JWT -> POST /v2/exclusions/range~batchGet -> Assert HTTP 200 OK. | Service account authorized. | GetByRangeV2 authorization. |
| 40 | | BatchGetExclusionsByRangeV2_Authorization (MissingJwt_Unauthorized) | Authorization: Missing JWT returns 401. | POST /v2/exclusions/range~batchGet no headers -> Assert HTTP 401 Unauthorized. | Unauthenticated rejected. | GetByRangeV2 authorization. |
| 41 | | BatchGetExclusionsByRangeV2_GivenNoEntityDateRanges_BadRequest | Validation: Empty entity date ranges returns 400. | POST /v2/exclusions/range~batchGet with empty list -> Assert HTTP 400 BadRequest. | Empty range list rejected. | GetByRangeV2 validation. |
| 42 | | BatchGetExclusionsByRangeV2_GivenTooManyEntityDateRanges_BadRequest | Validation: 505 entity ranges returns 400. | POST /v2/exclusions/range~batchGet with 505 ranges -> Assert HTTP 400 BadRequest. | Over-limit rejected. | GetByRangeV2 validation. |
| 43 | | BatchGetExclusionsByRangeV2_Success | Happy path: Create 2 exclusions, query by range including a non-existing entity, verify both found and fields match. | Create 2 exclusions with different entityIds -> POST /v2/exclusions/range~batchGet with 3 ranges (2 valid + 1 non-existing entity) and ExclusionType=Busy filter -> Assert 2 results -> Verify all fields via BeEquivalentTo. | Range query returns only matching exclusions with correct fields. | GetByRangeV2 success. In scope: multi-entity range query; type filter; field validation. Out of scope: pagination; no-match scenario. |
| 44 | | BatchGetExclusionsByRangeV2_OutOfRange | Range query with date range outside exclusion's time range returns empty. | Create exclusion -> POST /v2/exclusions/range~batchGet with date range +1 to +2 days from exclusion time -> Assert 0 results. | Exclusions outside the queried range are not returned. | GetByRangeV2 out-of-range. In scope: date range filtering. Out of scope: in-range scenarios. |

### GetExclusionsPage

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 45 | | GetExclusionsPage_PatientUser_Forbidden | Authorization: Patient user forbidden for paged exclusion query. | Get patient JWT -> POST /v1/exclusions/page -> Assert HTTP 403 Forbidden. | Patient users cannot access paged exclusion endpoint. | GetExclusionsPage auth. |
| 46 | | GetExclusionsPage_ProviderUser_Forbidden | Authorization: Provider user forbidden (service-account-only endpoint). | Get provider JWT -> POST /v1/exclusions/page -> Assert HTTP 403 Forbidden. | Provider users cannot access paged exclusion endpoint. | GetExclusionsPage auth. |
| 47 | | GetExclusionsPage_CsrUser_Forbidden | Authorization: CSR user forbidden. | Get CSR JWT -> POST /v1/exclusions/page -> Assert HTTP 403 Forbidden. | CSR users cannot access paged exclusion endpoint. | GetExclusionsPage auth. |
| 48 | | GetExclusionsPage_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | POST /v1/exclusions/page with no headers -> Assert HTTP 401 Unauthorized. | Unauthenticated rejected. | GetExclusionsPage auth. |
| 49 | | GetExclusionsPage_ServiceAccount_Success | Happy path: Service account can page through exclusions with type filter. | Create Busy + Appointment exclusions -> POST /v1/exclusions/page with ExclusionType=Appointment filter -> Assert Busy exclusion absent -> Assert Appointment exclusion present and equivalent -> Assert NextPaginationToken is null (all fit in one page). | Service account paged query with type filtering and single-page validation. | GetExclusionsPage success. In scope: type filtering; pagination token; service account auth. Out of scope: multi-page pagination. |

### BatchRemove

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 50 | | BatchRemoveExclusions_Authorization (PatientUser_Forbidden) | Authorization: Patient user cannot delete exclusions. | Get patient JWT -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 403 Forbidden. | Patient user forbidden. | BatchRemove auth. |
| 51 | | BatchRemoveExclusions_Authorization (ProviderUser_CorrectPracticeId_Ok) | Authorization: Provider with correct practice can delete. | Get provider JWT correct practice -> Add exclusions for all entities -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 200 OK. | Provider correct practice authorized. | BatchRemove auth. |
| 52 | | BatchRemoveExclusions_Authorization (ProviderUser_IncorrectPracticeId_Forbidden) | Authorization: Provider with wrong practice forbidden. | Get provider JWT wrong practice -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 403 Forbidden. | Provider wrong practice forbidden. | BatchRemove auth. |
| 53 | | BatchRemoveExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndRole_Ok) | Authorization: RBAC provider correct practice + PracticeFullAdmin OK. | Get RBAC JWT -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 200 OK. | RBAC authorized. | BatchRemove auth. |
| 54 | | BatchRemoveExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndIncorrectRole_Forbidden) | Authorization: RBAC correct practice + PracticeBilling forbidden. | Get RBAC JWT wrong role -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 403 Forbidden. | RBAC wrong role forbidden. | BatchRemove auth. |
| 55 | | BatchRemoveExclusions_Authorization (ProviderUser_IncorrectRbacPracticeIdAndCorrectRole_Forbidden) | Authorization: RBAC wrong practice + correct role forbidden. | Get RBAC JWT wrong practice -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 403 Forbidden. | RBAC wrong practice forbidden. | BatchRemove auth. |
| 56 | | BatchRemoveExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndPracticeAppointmentManagementRole_Ok) | Authorization: RBAC PracticeAppointmentManagement OK. | Get RBAC JWT -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 200 OK. | RBAC appointment management authorized. | BatchRemove auth. |
| 57 | | BatchRemoveExclusions_Authorization (CsrUser_WithoutReadWriteRole_Forbidden) | Authorization: CSR without ReadWrite cannot delete. | Get CSR JWT no role -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 403 Forbidden. | CSR without write role forbidden for deletes. | BatchRemove auth. |
| 58 | | BatchRemoveExclusions_Authorization (CsrUser_WithReadWriteRole_Ok) | Authorization: CSR with ReadWrite can delete. | Get CSR JWT with role -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 200 OK. | CSR with role authorized. | BatchRemove auth. |
| 59 | | BatchRemoveExclusions_Authorization (ServiceAccount_Ok) | Authorization: Service account can delete. | Get service account JWT -> POST /v1/exclusions/id~batchDelete -> Assert HTTP 200 OK. | Service account authorized. | BatchRemove auth. |
| 60 | | BatchRemoveExclusions_Authorization (MissingJwt_Unauthorized) | Authorization: Missing JWT returns 401. | POST /v1/exclusions/id~batchDelete no headers -> Assert HTTP 401 Unauthorized. | Unauthenticated rejected. | BatchRemove auth. |
| 61 | | BatchRemoveExclusions_GivenNoExclusions_BadRequest | Validation: Empty ID list returns 400. | POST /v1/exclusions/id~batchDelete with empty list -> Assert HTTP 400 BadRequest. | Empty list rejected. | BatchRemove validation. |
| 62 | | BatchRemoveExclusions_GivenTooManyExclusions_BadRequest | Validation: 105 IDs returns 400. | POST /v1/exclusions/id~batchDelete with 105 IDs -> Assert HTTP 400 BadRequest. | Over-limit rejected. | BatchRemove validation. |
| 63 | | BatchRemoveExclusions_Success | Happy path: Create exclusion, confirm via GetByIds and GetByRangeV2, delete, confirm absent from both. | Create exclusion -> Verify via GetByIds (1 result) and GetByRangeV2 (found) -> POST /v1/exclusions/id~batchDelete -> Verify GetByIds returns empty -> Verify GetByRangeV2 returns empty for that ID. | Full create-verify-delete-verify lifecycle. | BatchRemove success. In scope: delete; verify removal via GetByIds and GetByRangeV2. Out of scope: update. |

### GetHistory

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 64 | | BatchGetExclusionHistory_PatientUser_Forbidden [RealOnly] | Authorization: Patient user forbidden from viewing exclusion history. | Get patient JWT -> GET /v1/exclusion-history/{id} -> Assert HTTP 403 Forbidden. | Patient users cannot view history. | GetHistory auth. |
| 65 | | BatchGetExclusionHistory_ProviderUser_Forbidden [RealOnly] | Authorization: Provider user forbidden from viewing exclusion history. | Get provider JWT -> GET /v1/exclusion-history/{id} -> Assert HTTP 403 Forbidden. | Provider users cannot view history. | GetHistory auth. |
| 66 | | BatchGetExclusionHistory_CsrUser_Ok [RealOnly] | Authorization: CSR user CAN view exclusion history. | Get CSR JWT -> GET /v1/exclusion-history/{id} -> Assert HTTP 200 OK. | CSR users are authorized to view history. | GetHistory auth. |
| 67 | | BatchGetExclusionHistory_ServiceAccount_Forbidden [RealOnly] | Authorization: Service account forbidden from viewing history. | Get service account JWT -> GET /v1/exclusion-history/{id} -> Assert HTTP 403 Forbidden. | Service accounts cannot view history. | GetHistory auth. |
| 68 | | BatchGetExclusionHistory_MissingJwt_Unauthorized [RealOnly] | Authorization: Missing JWT returns 401. | GET /v1/exclusion-history/{id} no headers -> Assert HTTP 401 Unauthorized. | Unauthenticated rejected. | GetHistory auth. |
| 69 | | BatchGetExclusionHistory_Success [RealOnly] | Happy path: Create, update, delete an exclusion, then verify full history (3 entries with correct actions, fields, and chronological order). | Create Appointment exclusion -> Update (change EntityId, shift times +1h) -> Delete -> GET /v1/exclusion-history/{id} -> Assert 3 history entries -> Verify Create entry fields + action=Create + UpdatedBy + timestamp -> Verify Update entry fields + action=Update + updated values + timestamp > create -> Verify Delete entry fields + action=Delete + timestamp > update. | Full CRUD lifecycle history with chronological and field-level validation. | GetHistory success. In scope: create/update/delete history; field validation; chronological ordering; UpdatedBy tracking. Out of scope: pagination; partial history. |

### BatchUpdate

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 70 | | BatchUpdateExclusions_Authorization (PatientUser_Forbidden) | Authorization: Patient user cannot update exclusions. | Get patient JWT -> PUT /v1/exclusions -> Assert HTTP 403 Forbidden. | Patient user forbidden. | BatchUpdate auth. |
| 71 | | BatchUpdateExclusions_Authorization (ProviderUser_CorrectPracticeId_Ok) | Authorization: Provider with correct practice can update. | Get provider JWT correct practice -> PUT /v1/exclusions -> Assert HTTP 200 OK. | Provider correct practice authorized. | BatchUpdate auth. |
| 72 | | BatchUpdateExclusions_Authorization (ProviderUser_IncorrectPracticeId_Forbidden) | Authorization: Provider with wrong practice forbidden. | Get provider JWT wrong practice -> PUT /v1/exclusions -> Assert HTTP 403 Forbidden. | Provider wrong practice forbidden. | BatchUpdate auth. |
| 73 | | BatchUpdateExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndRole_Ok) | Authorization: RBAC correct practice + PracticeFullAdmin OK. | Get RBAC JWT -> PUT /v1/exclusions -> Assert HTTP 200 OK. | RBAC authorized. | BatchUpdate auth. |
| 74 | | BatchUpdateExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndIncorrectRole_Forbidden) | Authorization: RBAC correct practice + PracticeBilling forbidden. | Get RBAC JWT wrong role -> PUT /v1/exclusions -> Assert HTTP 403 Forbidden. | RBAC wrong role forbidden. | BatchUpdate auth. |
| 75 | | BatchUpdateExclusions_Authorization (ProviderUser_IncorrectRbacPracticeIdAndCorrectRole_Forbidden) | Authorization: RBAC wrong practice + correct role forbidden. | Get RBAC JWT wrong practice -> PUT /v1/exclusions -> Assert HTTP 403 Forbidden. | RBAC wrong practice forbidden. | BatchUpdate auth. |
| 76 | | BatchUpdateExclusions_Authorization (ProviderUser_CorrectRbacPracticeIdAndPracticeAppointmentManagementRole_Ok) | Authorization: RBAC PracticeAppointmentManagement OK. | Get RBAC JWT -> PUT /v1/exclusions -> Assert HTTP 200 OK. | RBAC appointment management authorized. | BatchUpdate auth. |
| 77 | | BatchUpdateExclusions_Authorization (CsrUser_WithoutReadWriteRole_Forbidden) | Authorization: CSR without ReadWrite cannot update. | Get CSR JWT no role -> PUT /v1/exclusions -> Assert HTTP 403 Forbidden. | CSR without write role forbidden. | BatchUpdate auth. |
| 78 | | BatchUpdateExclusions_Authorization (CsrUser_WithReadWriteRole_Ok) | Authorization: CSR with ReadWrite can update. | Get CSR JWT with role -> PUT /v1/exclusions -> Assert HTTP 200 OK. | CSR with role authorized. | BatchUpdate auth. |
| 79 | | BatchUpdateExclusions_Authorization (ServiceAccount_Ok) | Authorization: Service account can update. | Get service account JWT -> PUT /v1/exclusions -> Assert HTTP 200 OK. | Service account authorized. | BatchUpdate auth. |
| 80 | | BatchUpdateExclusions_Authorization (MissingJwt_Unauthorized) | Authorization: Missing JWT returns 401. | PUT /v1/exclusions no headers -> Assert HTTP 401 Unauthorized. | Unauthenticated rejected. | BatchUpdate auth. |
| 81 | | BatchUpdateExclusions_Success | Happy path: Create exclusion, update times (+1h), verify via GetByIds and GetByRangeV2. | Create Appointment exclusion -> Confirm created -> Update (shift StartDateTimeUtc and EndDateTimeUtc +1h) -> PUT /v1/exclusions -> GET by ID -> Assert updated fields match -> GET by range V2 with updated range -> Assert updated exclusion found and matches. | Full create-update-verify lifecycle. | BatchUpdate success. In scope: time field update; read-back via GetByIds and GetByRangeV2. Out of scope: entity change; delete. |
| 82 | | BatchUpdateExclusions_GivenNoExclusions_BadRequest | Validation: Empty list returns 400. | PUT /v1/exclusions with empty list -> Assert HTTP 400 BadRequest. | Empty list rejected. | BatchUpdate validation. |
| 83 | | BatchUpdateExclusions_GivenTooManyExclusions_BadRequest | Validation: 105 items returns 400. | PUT /v1/exclusions with 105 items -> Assert HTTP 400 BadRequest. | Over-limit rejected. | BatchUpdate validation. |

---

## tests/ApiTests/LeadTime/LeadTimeApiTests.cs

### AddOrUpdateLeadTime

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 84 | | AddOrUpdateLeadTime_PatientUser_Forbidden | Authorization: Patient user cannot add/update lead time. | Get patient JWT -> POST /v1/lead-time/practices/{practiceId}/entities -> Assert HTTP 403 Forbidden. | Patient user forbidden. | AddOrUpdate auth. |
| 85 | | AddOrUpdateLeadTime_PracticeUser_TestCases (CorrectPractice_SettingsManagement_Ok) | Authorization: Practice user with PracticeSettingsManagement role and correct practice can add lead time. | Get practice RBAC JWT correct practice + PracticeSettingsManagement -> POST lead time -> Assert HTTP 200 OK. | Practice user correct practice + role authorized. | AddOrUpdate auth. |
| 86 | | AddOrUpdateLeadTime_PracticeUser_TestCases (WrongPractice_Forbidden) | Authorization: Practice user with wrong practice is forbidden. | Get practice RBAC JWT wrong practice -> POST lead time -> Assert HTTP 403 Forbidden. | Practice user wrong practice forbidden. | AddOrUpdate auth. |
| 87 | | AddOrUpdateLeadTime_PracticeUser_TestCases (CorrectRbacPractice_SettingsManagement_Ok) | Authorization: RBAC user with correct rbacPracticeId + PracticeSettingsManagement OK. | Get practice RBAC JWT -> POST lead time -> Assert HTTP 200 OK. | RBAC correct practice + settings management authorized. | AddOrUpdate auth. |
| 88 | | AddOrUpdateLeadTime_PracticeUser_TestCases (WrongRbacPractice_Forbidden) | Authorization: RBAC user with wrong rbacPracticeId forbidden. | Get RBAC JWT wrong rbacPracticeId -> POST lead time -> Assert HTTP 403 Forbidden. | RBAC wrong practice forbidden. | AddOrUpdate auth. |
| 89 | | AddOrUpdateLeadTime_PracticeUser_TestCases (CorrectRbacPractice_WrongRole_Forbidden) | Authorization: RBAC user correct practice but PracticeBilling role forbidden. | Get RBAC JWT correct practice + PracticeBilling -> POST lead time -> Assert HTTP 403 Forbidden. | RBAC wrong role forbidden. | AddOrUpdate auth. |
| 90 | | AddOrUpdateLeadTime_CsrUser_TestCases (CsrPracticeEdit_Ok) | Authorization: CSR with CsrPracticeEdit role can add lead time. | Get CSR JWT CsrPracticeEdit -> POST lead time -> Assert HTTP 200 OK. | CSR with edit role authorized. | AddOrUpdate auth. |
| 91 | | AddOrUpdateLeadTime_CsrUser_TestCases (CsrUser_Forbidden) | Authorization: CSR with basic CsrUser role forbidden. | Get CSR JWT CsrUser -> POST lead time -> Assert HTTP 403 Forbidden. | Basic CSR forbidden. | AddOrUpdate auth. |
| 92 | | AddOrUpdateLeadTime_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | POST lead time with no headers -> Assert HTTP 401 Unauthorized. | Unauthenticated rejected. | AddOrUpdate auth. |
| 93 | | AddOrUpdateLeadTime_InvalidLeadTimeValue_BadRequest (Hour, 13) | Validation: Hour unit with value 13 (not an allowed option) returns 400. | POST lead time with unitType=Hour, value=13 -> Assert HTTP 400 -> Assert message "Lead time value: 13 is not allowed for the time unit: Hour" -> Assert errors contain "value" key. | Invalid hour values rejected with descriptive error. | AddOrUpdate validation. In scope: invalid Hour value. |
| 94 | | AddOrUpdateLeadTime_InvalidLeadTimeValue_BadRequest (Hour, 1) | Validation: Hour unit with value 1 (not allowed unless reward-eligible) returns 400. | POST lead time with unitType=Hour, value=1 -> Assert HTTP 400 -> Assert message about not allowed. | 1-hour lead time rejected for non-reward-eligible practices. | AddOrUpdate validation. In scope: restricted Hour value. |
| 95 | | AddOrUpdateLeadTime_InvalidEntityIdFormat_ReturnsBadRequest | Validation: Entity ID not matching expected format (pr_/lo_/pt_) returns 400. | POST lead time with entityId="invalid_entity_001" -> Assert HTTP 400 BadRequest. | Invalid entity ID format rejected. | AddOrUpdate validation. In scope: entity ID format validation. |
| 96 | | AddOrUpdateLeadTime_EntityId_NotFound | Validation: Non-existing entity ID returns 404. | POST lead time with entityId="lo_---------------------x", entityType=Location -> Assert HTTP 404 -> Assert message "Entity with id: lo_---------------------x of type: Location was not found". | Non-existing entity returns descriptive 404. | AddOrUpdate validation. In scope: entity not found. |
| 97 | | AddOrUpdateLeadTime_EntityIdNotInPractice_NotFound | Validation: Entity ID not belonging to the specified practice returns 404. | POST lead time with valid entityId but practiceId="pt_different" -> Assert HTTP 404 -> Assert message "Entity with id: pt_-------------------001 does not exist in practice_id: pt_different". | Cross-practice entity mismatch returns 404. | AddOrUpdate validation. In scope: entity-practice mismatch. |
| 98 | | AddOrUpdateLeadTime_MismatchedEntityType_BadRequest | Validation: Entity ID prefix doesn't match declared entity type (pt_ ID with Location type) returns 400. | POST lead time with entityId=pt_... but entityType=Location -> Assert HTTP 400 -> Assert message "Entity with id: pt_-------------------001 does not match entity type: Location" -> Assert errors contain "entityType". | Entity ID and type mismatch rejected. | AddOrUpdate validation. In scope: entity type mismatch. |
| 99 | | AddOrUpdateLeadTime_CanCreateThenUpdate_Success | Happy path: Create lead time (Hour, 2), verify response, update to value=4, verify updated response. | POST lead time value=2 -> Assert response matches (EntityId, EntityType=Practice, UnitType=Hour, Value=2) -> POST again value=4 -> Assert response matches Value=4. | Create and upsert lead time with response field validation. | AddOrUpdate success. In scope: create; update; response validation. Out of scope: delete; get. |
| 100 | | AddOrUpdateLeadTime_CanCreateRewardValue_Success [RealOnly] | Happy path: Reward-eligible practice can create lead time with Hour value=1. | Seed reward-eligible practice via ProviderReferenceSeeder -> POST lead time with practiceId=reward-eligible, unitType=Hour, value=1 -> Assert response matches (Value=1). | Reward-eligible practices can use the 1-hour lead time option. | AddOrUpdate success. In scope: reward-eligible practice; 1-hour value. Out of scope: non-eligible practices. |

### DeleteLeadTime

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 101 | | DeleteLeadTime_PatientUser_Forbidden | Authorization: Patient user cannot delete lead time. | Get patient JWT -> DELETE /v1/lead-time/practices/{id}/entities -> Assert HTTP 403 Forbidden. | Patient forbidden. | Delete auth. |
| 102 | | DeleteLeadTime_PracticeUser_TestCases (5 cases) | Authorization: Practice RBAC user authorization for delete (same matrix as add/update). | Get RBAC JWT with various practice/role combos -> DELETE lead time -> Assert expected status code. | Practice RBAC authorization matrix for delete. | Delete auth. |
| 103 | | DeleteLeadTime_CsrUser_TestCases (2 cases) | Authorization: CSR role matrix for delete. | Get CSR JWT (CsrPracticeEdit=OK, CsrUser=Forbidden) -> DELETE lead time -> Assert expected status code. | CSR role authorization for delete. | Delete auth. |
| 104 | | DeleteLeadTime_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | DELETE lead time no headers -> Assert HTTP 401 Unauthorized. | Unauthenticated rejected. | Delete auth. |
| 105 | | DeleteProviderLeadTime_WithValidEntityId_DeletesSpecificLeadTime [RealOnly] | Happy path: Create provider lead time, delete it, verify gone via batch get. | Seed provider -> Create lead time for pr_1 -> Verify response -> DELETE /v1/lead-time/practices/{id}/entities?entity_id=pr_1 -> Assert HTTP 200 -> GET lead times for Provider type -> Assert empty. | Provider-level lead time delete with verification. | Delete success. In scope: create; delete; verify absence. |
| 106 | | DeleteLeadTime_InvalidEntityIdFormat_ReturnsBadRequest | Validation: Invalid entity ID format returns 400. | DELETE with entityId="invalid_entity_001" -> Assert HTTP 400 BadRequest. | Invalid entity ID format rejected. | Delete validation. |
| 107 | | DeleteLeadTime_EntityIdNotInPractice_NotFound [RealOnly] | Validation: Entity not in specified practice returns 404. | Create lead time for pr_1 in practice -> DELETE with practiceId="pt_different" -> Assert HTTP 404 -> Assert message about entity not in practice. | Cross-practice entity mismatch returns 404. | Delete validation. |

### BatchGetLeadTimesForPractice

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 108 | | BatchGetLeadTimesForPractice_PatientUser_Forbidden | Authorization: Patient user forbidden. | Get patient JWT -> GET lead times -> Assert HTTP 403 Forbidden. | Patient forbidden. | BatchGet auth. |
| 109 | | BatchGetLeadTimesForPractice_PracticeUser_TestCases (5 cases) | Authorization: Practice RBAC role matrix for get. | Get RBAC JWT various combos -> GET lead times -> Assert expected status code. | RBAC authorization matrix for get. | BatchGet auth. |
| 110 | | BatchGetLeadTimesForPractice_CsrUser_TestCases (2 cases) | Authorization: CSR role matrix for get. | Get CSR JWT -> GET lead times -> Assert expected status code. | CSR authorization for get. | BatchGet auth. |
| 111 | | BatchGetLeadTimesForPractice_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | GET lead times no headers -> Assert HTTP 401. | Unauthenticated rejected. | BatchGet auth. |
| 112 | | BatchGetLeadTimesForPractice_NoEntityTypes_BadRequest | Validation: Empty entity_types query param returns 400. | GET lead times with empty entity_types -> Assert HTTP 400 BadRequest. | Empty entity types rejected. | BatchGet validation. |
| 113 | | BatchGetLeadTimesForPractice_AddAndGetLeadTime_ReturnsSuccess | Happy path: Create lead time then retrieve via batch get, verify match. | POST lead time (Hour, 2) -> Verify response -> GET /v1/lead-time/practices/{id}?entity_types=Practice -> Assert 1 result -> Assert result matches created. | Create and batch-get lifecycle. | BatchGet success. In scope: create; get; field match. |

### GetPracticeLeadTimeOptions

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 114 | | GetPracticeLeadTimeOptions_PatientUser_Forbidden | Authorization: Patient user forbidden. | Get patient JWT -> GET options -> Assert HTTP 403 Forbidden. | Patient forbidden. | GetOptions auth. |
| 115 | | GetPracticeLeadTimeOptions_PracticeUser_TestCases (5 cases) | Authorization: Practice RBAC role matrix for options. | Get RBAC JWT -> GET options -> Assert expected status code. | RBAC auth matrix. | GetOptions auth. |
| 116 | | GetPracticeLeadTimeOptions_CsrUser_TestCases (2 cases) | Authorization: CSR role matrix for options. | Get CSR JWT -> GET options -> Assert expected status code. | CSR auth matrix. | GetOptions auth. |
| 117 | | GetPracticeLeadTimeOptions_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | GET options no headers -> Assert HTTP 401. | Unauthenticated rejected. | GetOptions auth. |
| 118 | | GetPracticeLeadTimeOptions_ReturnsAllDefaultOptions | Happy path: Returns 8 default lead time options (2h/4h/6h/1d-5d). | GET /v1/lead-time/practices/{id}/options -> Assert 8 options -> Verify exact list: Hour(2,4,6), Day(1,2,3,4,5). | Default options list validation. | GetOptions success. In scope: default option list. Out of scope: reward options. |
| 119 | | GetPracticeLeadTimeOptions_ReturnsOptionsWithReward | Happy path: Reward-eligible practice returns 9 options (includes Hour value=1). | GET /v1/lead-time/practices/{rewardPracticeId}/options -> Assert 9 options -> Verify exact list includes Hour(1). | Reward-eligible practice gets additional 1-hour option. | GetOptions success. In scope: reward-eligible options. |

---

## tests/ApiTests/LeadTime/BusinessHoursApiTests.cs

### AddOrUpdate Business Hours

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 120 | | AddOrUpdateForPractice_CanCreateThenUpdate_Success | Happy path: Create business hours (Monday 9-17), then update to Tuesday 10-18, verify both responses. | PUT /v1/business-hours/practices/{id} with Monday 9:00-17:00 -> Assert 1 day, Monday, 1 segment -> PUT again with Tuesday 10:00-18:00 -> Assert 1 day, Tuesday. | Create and update business hours with field validation. | AddOrUpdate success. In scope: create; update; day/segment verification. |
| 121 | | AddOrUpdateForPractice_CsrUser_TestCases (CsrPracticeEdit_Ok) | Authorization: CSR with CsrPracticeEdit can manage business hours. | Get CSR JWT CsrPracticeEdit -> PUT business hours -> Assert HTTP 200 OK. | CSR with edit role authorized. | AddOrUpdate auth. |
| 122 | | AddOrUpdateForPractice_CsrUser_TestCases (CsrUser_Forbidden) | Authorization: Basic CSR user forbidden. | Get CSR JWT CsrUser -> PUT business hours -> Assert HTTP 403 Forbidden. | Basic CSR forbidden. | AddOrUpdate auth. |
| 123 | | AddOrUpdateForPractice_EmptySegments_BadRequest | Validation: Empty segments list returns 400 with message "You must provide at least one business hour segment". | PUT business hours with empty segments -> Assert HTTP 400 -> Assert error message. | Empty segments rejected with descriptive error. | AddOrUpdate validation. |
| 124 | | AddOrUpdateForPractice_EndTimeBeforeStartTime_BadRequest | Validation: End time before start time returns 400. | PUT business hours startTime=17:00, endTime=09:00 -> Assert HTTP 400 -> Assert message "End time 09:00 must be after start time 17:00". | Time order validation. | AddOrUpdate validation. |
| 125 | | AddOrUpdateForPractice_InvalidTimeFormat_BadRequest (7 cases) | Validation: Various invalid time formats (invalid_time, 25:00, abc:def, HH:mm:ss) return 400. | PUT business hours with invalid time strings -> Assert HTTP 400 -> Assert validation errors on StartTime or EndTime. | Time format validation across multiple invalid patterns. | AddOrUpdate validation. |
| 126 | | AddOrUpdateForPractice_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | PUT business hours no headers -> Assert HTTP 401. | Unauthenticated rejected. | AddOrUpdate auth. |
| 127 | | AddOrUpdateForPractice_MultipleDays_Success | Happy path: Create business hours with Monday and Tuesday, verify 2 days returned. | PUT business hours with Monday + Tuesday -> Assert 2 SegmentsByDay entries. | Multiple days creation. | AddOrUpdate success. |
| 128 | | AddOrUpdateForPractice_MultipleSegmentsPerDay_BadRequest | Validation: Multiple segments per day returns 400 with "You can only have one segment per day". | PUT business hours with Monday (09:00-12:00, 13:00-17:00) -> Assert HTTP 400 -> Assert error message. | Single-segment-per-day constraint enforced. | AddOrUpdate validation. |
| 129 | | AddOrUpdateForPractice_PatientUser_Forbidden | Authorization: Patient user forbidden. | Get patient JWT -> PUT business hours -> Assert HTTP 403 Forbidden. | Patient forbidden. | AddOrUpdate auth. |
| 130 | | AddOrUpdateForPractice_PracticeUser_TestCases (5 cases) | Authorization: Practice RBAC role matrix with PracticeSettingsManagement role. | Get RBAC JWT various combos -> PUT business hours -> Assert expected status code. | RBAC auth matrix. | AddOrUpdate auth. |
| 131 | | AddOrUpdateForPractice_WithDuplicateSegments_BadRequest | Validation: Duplicate segments returns "You can only have one segment per day". | PUT business hours with 5 segments (including duplicates) -> Assert HTTP 400. | Duplicate segment detection. | AddOrUpdate validation. |
| 132 | | AddOrUpdateForPractice_WithMultipleDaysAndDuplicates_BadRequest | Validation: Multiple days with duplicate days and segments returns 400. | PUT business hours with Monday (5 segments with dupes) + Tue/Wed/Thu (with duplicate days) -> Assert HTTP 400 -> Assert "You can only have one segment per day". | Duplicate day + segment detection. | AddOrUpdate validation. |

### Delete Business Hours

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 133 | | DeleteForPractice_CsrUser_TestCases (CsrPracticeEdit_NoContent) | Authorization: CSR with CsrPracticeEdit can delete. | Get CSR JWT -> DELETE business hours -> Assert HTTP 204 NoContent. | CSR authorized for delete. | Delete auth. |
| 134 | | DeleteForPractice_CsrUser_TestCases (CsrUser_Forbidden) | Authorization: Basic CSR forbidden for delete. | Get CSR JWT CsrUser -> DELETE -> Assert HTTP 403 Forbidden. | Basic CSR forbidden. | Delete auth. |
| 135 | | DeleteForPractice_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | DELETE business hours no headers -> Assert HTTP 401. | Unauthenticated rejected. | Delete auth. |
| 136 | | DeleteForPractice_PatientUser_Forbidden | Authorization: Patient forbidden. | Get patient JWT -> DELETE -> Assert HTTP 403 Forbidden. | Patient forbidden. | Delete auth. |
| 137 | | DeleteForPractice_PracticeUser_TestCases (5 cases) | Authorization: RBAC matrix for delete (NoContent for authorized, Forbidden otherwise). | Get RBAC JWT -> DELETE -> Assert expected status code. | RBAC auth matrix for delete. | Delete auth. |
| 138 | | DeleteForPractice_WithNonExistentPractice_Success | Idempotency: Deleting non-existent practice returns 204 NoContent. | DELETE /v1/business-hours/practices/pt_nonexistent -> Assert HTTP 204 NoContent. | Delete is idempotent. | Delete idempotency. |
| 139 | | DeleteForPractice_WithValidPracticeId_Success | Happy path: Create then delete business hours, verify 204. | PUT business hours -> DELETE -> Assert HTTP 204 NoContent. | Create-delete lifecycle. | Delete success. |

### Get Business Hours

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 140 | | GetForPractice_CsrUser_TestCases (2 cases) | Authorization: CSR role matrix for get (CsrPracticeEdit=NoContent, CsrUser=Forbidden). | Get CSR JWT -> GET business hours -> Assert expected status code. | CSR auth for get. | Get auth. |
| 141 | | GetForPractice_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | GET business hours no headers -> Assert HTTP 401. | Unauthenticated rejected. | Get auth. |
| 142 | | GetForPractice_PatientUser_Forbidden | Authorization: Patient forbidden. | Get patient JWT -> GET -> Assert HTTP 403 Forbidden. | Patient forbidden. | Get auth. |
| 143 | | GetForPractice_PracticeUser_TestCases (5 cases) | Authorization: RBAC matrix for get. | Get RBAC JWT -> GET -> Assert expected status code. | RBAC auth matrix for get. | Get auth. |
| 144 | | GetForPractice_WithNonExistentPractice_NoContent | No business hours for non-existent practice returns 204 NoContent. | GET /v1/business-hours/practices/pt_nonexistent -> Assert HTTP 204 NoContent. | Non-existent practice returns NoContent. | Get edge case. |

---

## tests/ApiTests/Templates/AvailabilityTemplatesAuthApiTests.cs

### Authorization Tests for All Template Operations

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 145 | | CreateTemplatedAvailability_PatientUser_Forbidden | Authorization: Patient cannot create templates. | Get patient JWT -> POST /v1/templated-availability/practices/{id} -> Assert HTTP 403. | Patient forbidden for create. | Create auth. |
| 146 | | CreateTemplatedAvailability_PracticeUser_TestCases (5 cases) | Authorization: Practice RBAC matrix for create (PracticeAppointmentManagement=Created, wrong practice/role=Forbidden). | Get RBAC JWT -> POST template -> Assert expected status (201 Created or 403 Forbidden). | RBAC auth matrix for create. | Create auth. |
| 147 | | CreateTemplatedAvailability_CsrUser_TestCases (3 cases) | Authorization: CSR role matrix for create (CsrPracticeEdit=Created, CsrCanModifyTimesInProviderCalendar=Created, CsrUser=Forbidden). | Get CSR JWT -> POST template -> Assert expected status. | CSR auth matrix for create. | Create auth. |
| 148 | | CreateTemplatedAvailability_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | POST template no headers -> Assert HTTP 401. | Unauthenticated rejected. | Create auth. |
| 149 | | GetTemplatedAvailabilityById_PatientUser_Forbidden | Authorization: Patient cannot get template by ID. | Get patient JWT -> GET template by ID -> Assert HTTP 403. | Patient forbidden for get-by-id. | GetById auth. |
| 150 | | GetTemplatedAvailabilityById_PracticeUser_TestCases (5 cases) | Authorization: Practice RBAC matrix for get-by-id (authorized returns 404 since template doesn't exist, forbidden returns 403). | Get RBAC JWT -> GET template -> Assert expected status (404 NotFound or 403 Forbidden). | RBAC auth matrix for get-by-id. | GetById auth. |
| 151 | | GetTemplatedAvailabilityById_CsrUser | Authorization: CSR user can get template by ID (returns 404 since it doesn't exist). | Get CSR JWT CsrUser -> GET template -> Assert HTTP 404. | CSR authorized for read. | GetById auth. |
| 152 | | GetTemplatedAvailabilityById_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | GET template no headers -> Assert HTTP 401. | Unauthenticated rejected. | GetById auth. |
| 153 | | UpdateTemplatedAvailability_PatientUser_Forbidden | Authorization: Patient cannot update templates. | Get patient JWT -> PUT template -> Assert HTTP 403. | Patient forbidden for update. | Update auth. |
| 154 | | UpdateTemplatedAvailability_PracticeUser_TestCases (5 cases) | Authorization: Practice RBAC matrix for update (authorized returns 404, forbidden returns 403). | Get RBAC JWT -> PUT template -> Assert expected status. | RBAC auth matrix for update. | Update auth. |
| 155 | | UpdateTemplatedAvailability_CsrUser_TestCases (3 cases) | Authorization: CSR role matrix for update (CsrPracticeEdit/CsrCanModifyTimes=NotFound, CsrUser=Forbidden). | Get CSR JWT -> PUT template -> Assert expected status. | CSR auth matrix for update. | Update auth. |
| 156 | | UpdateTemplatedAvailability_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | PUT template no headers -> Assert HTTP 401. | Unauthenticated rejected. | Update auth. |
| 157 | | DeleteTemplatedAvailability_PatientUser_Forbidden | Authorization: Patient cannot delete templates. | Get patient JWT -> DELETE template -> Assert HTTP 403. | Patient forbidden for delete. | Delete auth. |
| 158 | | DeleteTemplatedAvailability_PracticeUser_TestCases (5 cases) | Authorization: Practice RBAC matrix for delete. | Get RBAC JWT -> DELETE template -> Assert expected status. | RBAC auth matrix for delete. | Delete auth. |
| 159 | | DeleteTemplatedAvailability_CsrUser_TestCases (3 cases) | Authorization: CSR role matrix for delete. | Get CSR JWT -> DELETE template -> Assert expected status. | CSR auth matrix for delete. | Delete auth. |
| 160 | | DeleteTemplatedAvailability_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | DELETE template no headers -> Assert HTTP 401. | Unauthenticated rejected. | Delete auth. |
| 161 | | BackfillTemplatedAvailability_PatientUser_Forbidden | Authorization: Patient cannot backfill. | Get patient JWT -> PUT backfill -> Assert HTTP 403. | Patient forbidden for backfill. | Backfill auth. |
| 162 | | BackfillTemplatedAvailability_PracticeUser_Forbidden | Authorization: Practice user (even with PracticeAppointmentManagement) cannot backfill. | Get RBAC JWT -> PUT backfill -> Assert HTTP 403. | Practice user forbidden for backfill. | Backfill auth. |
| 163 | | BackfillTemplatedAvailability_CsrUser_TestCases (3 cases) | Authorization: CSR role matrix for backfill (CsrDeveloper=NoContent, CsrPracticeEdit=Forbidden, CsrUser=Forbidden). | Get CSR JWT -> PUT backfill -> Assert expected status. | CSR auth matrix for backfill. | Backfill auth. |
| 164 | | BackfillTemplatedAvailability_ServiceRole_Success | Authorization: Service role (InteropAvailabilityApiExclusionsReadWrite) can backfill. | Get service JWT -> PUT backfill -> Assert HTTP 204 NoContent. | Service role authorized for backfill. | Backfill auth. |
| 165 | | BackfillTemplatedAvailability_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | PUT backfill no headers -> Assert HTTP 401. | Unauthenticated rejected. | Backfill auth. |
| 166 | | SearchTemplatedAvailability_PatientUser_Forbidden | Authorization: Patient cannot search templates. | Get patient JWT -> POST search -> Assert HTTP 403. | Patient forbidden for search. | Search auth. |
| 167 | | SearchTemplatedAvailability_PracticeUser_Forbidden | Authorization: Practice user cannot search (service-only). | Get RBAC JWT -> POST search -> Assert HTTP 403. | Practice user forbidden for search. | Search auth. |
| 168 | | SearchTemplatedAvailability_CsrUser_TestCases | Authorization: CSR user can search templates. | Get CSR JWT CsrUser -> POST search -> Assert HTTP 200 OK. | CSR authorized for search. | Search auth. |
| 169 | | SearchTemplatedAvailability_ServiceRole_Success | Authorization: Service role (InteropAvailabilityApiTimeSlotsRead) can search. | Get service JWT -> POST search -> Assert HTTP 200 OK. | Service role authorized for search. | Search auth. |
| 170 | | SearchTemplatedAvailability_MissingJwt_Unauthorized | Authorization: Missing JWT returns 401. | POST search no headers -> Assert HTTP 401. | Unauthenticated rejected. | Search auth. |

---

## tests/ApiTests/Templates/AvailabilityTemplatesCreateApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 171 | | CreateTemplatedAvailability_DayOnly_ValidRequest_Success | Happy path: Create DayOnly template, verify auto-generated ID and all fields via get-by-id. | POST template (DayOnly, tomorrow, 9:00-17:00, NY timezone) -> Assert TemplateId starts with "at_" -> GET by ID -> Assert all fields match (provider, location, dates, times, timezone, schedule type, patient type, practice). | Full DayOnly create with field validation. | Create success DayOnly. |
| 172 | | CreateTemplatedAvailability_Weekly_ValidRequest_Success | Happy path: Create Weekly recurring template, verify all fields. | POST template (Weekly, Mon/Wed/Fri, 9:00-17:00) -> GET by ID -> Assert all fields match including DaysOfWeek. | Full Weekly create with field validation. | Create success Weekly. |
| 173 | | CreateTemplatedAvailability_EveryOtherWeek_ValidRequest_Success | Happy path: Create EveryOtherWeek template, verify all fields. | POST template (EveryOtherWeek, Tue/Thu) -> GET by ID -> Assert all fields match. | Full EveryOtherWeek create with field validation. | Create success EveryOtherWeek. |
| 174 | | CreateTemplatedAvailability_Weekly_WithEndDate_Success | Happy path: Create Weekly template with explicit end date. | POST template (Weekly with EndDate = StartDate+30) -> GET by ID -> Assert EndDate matches. | Weekly template with end date. | Create success Weekly with EndDate. |
| 175 | | GetTemplatedAvailabilityById_TemplateNotFound_ReturnsNotFound | Error handling: Non-existent template ID returns 404. | GET /v1/templated-availability/practices/{id}/templates/at_nonexistent -> Assert HTTP 404. | Template not found handling. | GetById edge case. |
| 176 | | GetTemplatedAvailabilityById_WrongPracticeId_ReturnsNotFound | Error handling: Template exists but queried with wrong practice ID returns 404. | Create template -> GET with different practiceId -> Assert HTTP 404. | Cross-practice isolation. | GetById edge case. |
| 177 | | GetTemplatedAvailabilityById_ValidTemplate_ReturnsTemplate | Happy path: Create and retrieve template by ID. | Create template -> GET by ID -> Assert non-null, ID matches, ProviderId/LocationId/PracticeId match. | Basic get-by-id success. | GetById success. |
| 178 | | CreateTemplatedAvailability_DayOnly_MissingEndDate_BadRequest | Validation: DayOnly with null EndDate returns 400 "End date is required". | POST DayOnly template EndDate=null -> Assert HTTP 400 -> Assert errors["end_date"] contains "End date is required". | DayOnly EndDate required. | Create validation DayOnly. |
| 179 | | CreateTemplatedAvailability_DayOnly_EndDateDifferentFromStartDate_BadRequest | Validation: DayOnly with EndDate != StartDate returns 400. | POST DayOnly template EndDate=StartDate+1 -> Assert HTTP 400 -> Assert "must be the same as start date". | DayOnly date equality constraint. | Create validation DayOnly. |
| 180 | | CreateTemplatedAvailability_DayOnly_WithDaysOfWeek_BadRequest | Validation: DayOnly with DaysOfWeek set returns 400. | POST DayOnly template DaysOfWeek=[Monday] -> Assert HTTP 400 -> Assert "must not be provided for day-only templates". | DayOnly should not have DaysOfWeek. | Create validation DayOnly. |
| 181 | | CreateTemplatedAvailability_DayOnly_StartDateInPast_BadRequest | Validation: DayOnly with past StartDate returns 400. | POST DayOnly template StartDate=yesterday -> Assert HTTP 400 -> Assert "cannot be in the past". | Past date rejection. | Create validation DayOnly. |
| 182 | | CreateTemplatedAvailability_DayOnly_InvalidTimezone_BadRequest | Validation: Invalid IANA timezone returns 400. | POST DayOnly template IanaTimezone="Invalid/Timezone" -> Assert HTTP 400 -> Assert errors contain "iana_timezone". | Timezone validation. | Create validation DayOnly. |
| 183 | | CreateTemplatedAvailability_DayOnly_FromTimeEqualsToTime_BadRequest | Validation: FromTime == ToTime returns 400. | POST DayOnly template FromTime=ToTime=09:00 -> Assert HTTP 400 -> Assert errors on both "from_time" and "to_time" with "cannot be the same". | Equal time rejection. | Create validation DayOnly. |
| 184 | | CreateTemplatedAvailability_DayOnly_FromTimeAfterToTime_BadRequest | Validation: FromTime > ToTime returns 400. | POST DayOnly template FromTime=17:00 ToTime=09:00 -> Assert HTTP 400 -> Assert "must be before end time". | Time order validation. | Create validation DayOnly. |
| 185 | | CreateTemplatedAvailability_Weekly_MissingDaysOfWeek_BadRequest | Validation: Weekly with null DaysOfWeek returns 400. | POST Weekly template DaysOfWeek=null -> Assert HTTP 400 -> Assert "Days of week are required". | Weekly DaysOfWeek required. | Create validation Weekly. |
| 186 | | CreateTemplatedAvailability_Weekly_EmptyDaysOfWeek_BadRequest | Validation: Weekly with empty DaysOfWeek returns 400. | POST Weekly template DaysOfWeek=[] -> Assert HTTP 400 -> Assert "Days of week are required". | Weekly empty DaysOfWeek rejection. | Create validation Weekly. |
| 187 | | CreateTemplatedAvailability_Weekly_EndDateBeforeStartDate_BadRequest | Validation: Weekly EndDate < StartDate returns 400. | POST Weekly template EndDate=StartDate-1 -> Assert HTTP 400 -> Assert "must be greater than or equal to start date". | Date range validation. | Create validation Weekly. |
| 188 | | CreateTemplatedAvailability_Weekly_StartDateInPast_BadRequest | Validation: Weekly with past StartDate returns 400. | POST Weekly template StartDate=yesterday -> Assert HTTP 400 -> Assert "cannot be in the past". | Past date rejection. | Create validation Weekly. |
| 189 | | CreateTemplatedAvailability_Weekly_InvalidTimezone_BadRequest | Validation: Weekly with invalid timezone returns 400. | POST Weekly template IanaTimezone="Invalid/Timezone" -> Assert HTTP 400 -> errors contain "iana_timezone". | Timezone validation. | Create validation Weekly. |
| 190 | | CreateTemplatedAvailability_Weekly_FromTimeEqualsToTime_BadRequest | Validation: Weekly FromTime == ToTime returns 400. | POST Weekly template FromTime=ToTime -> Assert HTTP 400 -> errors on both time fields. | Equal time rejection. | Create validation Weekly. |
| 191 | | CreateTemplatedAvailability_Weekly_FromTimeAfterToTime_BadRequest | Validation: Weekly FromTime > ToTime returns 400. | POST Weekly template FromTime=17:00, ToTime=09:00 -> Assert HTTP 400 -> "must be before end time". | Time order validation. | Create validation Weekly. |
| 192 | | CreateTemplatedAvailability_EveryOtherWeek_MissingDaysOfWeek_BadRequest | Validation: EveryOtherWeek with null DaysOfWeek returns 400. | POST EveryOtherWeek template DaysOfWeek=null -> Assert HTTP 400. | DaysOfWeek required for EveryOtherWeek. | Create validation EveryOtherWeek. |
| 193 | | CreateTemplatedAvailability_EveryOtherWeek_EndDateBeforeStartDate_BadRequest | Validation: EveryOtherWeek EndDate < StartDate returns 400. | POST EveryOtherWeek template EndDate < StartDate -> Assert HTTP 400. | Date range validation. | Create validation EveryOtherWeek. |

---

## tests/ApiTests/Templates/AvailabilityTemplatesUpdateApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 194 | | UpdateTemplatedAvailability_DayOnly_ValidRequest_Success | Happy path: Create DayOnly template, update dates and times, verify TemplateId preserved and fields updated. | Create DayOnly template -> Update with new StartDate/EndDate (+5 days), FromTime=10:00, ToTime=18:00 -> Assert TemplateId preserved -> GET by ID -> Assert all fields match updated values. | DayOnly update with field validation. | Update success DayOnly. |
| 195 | | UpdateTemplatedAvailability_Weekly_ValidRequest_Success | Happy path: Create Weekly template, update DaysOfWeek and StartDate, verify. | Create Weekly (Mon/Wed/Fri) -> Update to Tue/Thu, StartDate+5 -> Assert TemplateId preserved -> GET by ID -> Assert fields match. | Weekly update with field validation. | Update success Weekly. |
| 196 | | UpdateTemplatedAvailability_DayOnly_MissingEndDate_BadRequest | Validation: Update DayOnly with null EndDate returns 400. | Create DayOnly -> Update with EndDate=null -> Assert HTTP 400 -> Assert "End date is required". | DayOnly EndDate required on update. | Update validation. |
| 197 | | UpdateTemplatedAvailability_DayOnly_EndDateDifferentFromStartDate_BadRequest | Validation: Update DayOnly EndDate != StartDate returns 400. | Create DayOnly -> Update EndDate=StartDate+1 -> Assert HTTP 400 -> Assert "must be the same as start date". | DayOnly date equality on update. | Update validation. |
| 198 | | UpdateTemplatedAvailability_WithStartDateInPastNotMatchingExisting_BadRequest | Validation: Update to past StartDate (different from existing) returns 400. | Create with future date -> Update StartDate to 5 days ago -> Assert HTTP 400 -> Assert "cannot be in the past". | Past date rejection on update (when changing). | Update validation. |
| 199 | | UpdateTemplatedAvailability_Weekly_MissingDaysOfWeek_BadRequest | Validation: Update Weekly with null DaysOfWeek returns 400. | Create Weekly -> Update DaysOfWeek=null -> Assert HTTP 400 -> Assert "Days of week are required". | DaysOfWeek required on Weekly update. | Update validation. |
| 200 | | UpdateTemplatedAvailability_Weekly_EndDateBeforeStartDate_BadRequest | Validation: Update Weekly EndDate < StartDate returns 400. | Create Weekly -> Update EndDate < StartDate -> Assert HTTP 400 -> Assert "must be greater than or equal to start date". | Date range validation on update. | Update validation. |
| 201 | | UpdateTemplatedAvailability_InvalidTimezone_BadRequest | Validation: Update with invalid timezone returns 400. | Create DayOnly -> Update IanaTimezone="Invalid/Timezone" -> Assert HTTP 400 -> errors contain "iana_timezone". | Timezone validation on update. | Update validation. |
| 202 | | UpdateTemplatedAvailability_TemplateNotFound_ReturnsNotFound | Error handling: Update non-existent template returns 404. | PUT /templates/at_nonexistent -> Assert HTTP 404. | Template not found on update. | Update edge case. |
| 203 | | UpdateTemplatedAvailability_WrongPracticeId_ReturnsNotFound | Error handling: Update template with wrong practiceId returns 404. | Create template -> PUT with different practiceId -> Assert HTTP 404. | Cross-practice isolation on update. | Update edge case. |

---

## tests/ApiTests/Templates/AvailabilityTemplatesDeleteApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 204 | | DeleteTemplatedAvailability_ValidTemplate_Success | Happy path: Create template, delete it, verify it's gone (404 on get). | Create DayOnly template -> DELETE -> GET by ID -> Assert HTTP 404 NotFound. | Full create-delete-verify lifecycle. | Delete success. |
| 205 | | DeleteTemplatedAvailability_TemplateNotFound_ReturnsNotFound | Error handling: Delete non-existent template returns 404. | DELETE /templates/at_nonexistent -> Assert HTTP 404. | Template not found on delete. | Delete edge case. |
| 206 | | DeleteTemplatedAvailability_WrongPracticeId_ReturnsNotFound | Error handling: Delete template with wrong practiceId returns 404. | Create template -> DELETE with different practiceId -> Assert HTTP 404. | Cross-practice isolation on delete. | Delete edge case. |

---

## tests/ApiTests/Templates/AvailabilityTemplatesBackfillApiTests.cs

### Happy Path

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 207 | | BackfillTemplatedAvailability_DayOnly_ValidRequest_Success | Happy path: Backfill a DayOnly template, retrieve and verify. | PUT backfill with DayOnly template -> GET by ID -> Assert TemplateId, PracticeId, ScheduleType=DayOnly. | DayOnly backfill creation. | Backfill success DayOnly. |
| 208 | | BackfillTemplatedAvailability_Weekly_ValidRequest_Success | Happy path: Backfill a Weekly template, retrieve and verify DaysOfWeek. | PUT backfill with Weekly template -> GET by ID -> Assert ScheduleType=Weekly, DaysOfWeek contains Monday. | Weekly backfill creation. | Backfill success Weekly. |
| 209 | | BackfillTemplatedAvailability_UpsertExistingTemplate_Success | Happy path: Backfill upserts an existing template (created via normal create). | Create template via POST -> Backfill same templateId with new dates/times -> GET by ID -> Assert StartDate, FromTime, ToTime reflect backfill values. | Backfill upsert behavior. | Backfill upsert. |
| 210 | | BackfillTemplatedAvailability_MultipleTemplates_Success | Happy path: Backfill 3 templates (DayOnly, Weekly, EveryOtherWeek) at once. | PUT backfill with 3 templates -> GET each by ID -> Assert ScheduleType matches for each. | Multi-template backfill. | Backfill success multiple. |

### DayOnly Bad Request Tests

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 211 | | BackfillTemplatedAvailability_DayOnly_MissingEndDate_BadRequest | Validation: DayOnly backfill with null EndDate returns 400. | PUT backfill DayOnly EndDate=null -> Assert HTTP 400 -> errors["end_date"] "End date is required". | EndDate required for DayOnly. | Backfill validation DayOnly. |
| 212 | | BackfillTemplatedAvailability_DayOnly_EndDateDifferentFromStartDate_BadRequest | Validation: DayOnly backfill EndDate != StartDate returns 400. | PUT backfill DayOnly EndDate=StartDate+1 -> Assert HTTP 400 -> "must be the same as start date". | DayOnly date equality. | Backfill validation DayOnly. |
| 213 | | BackfillTemplatedAvailability_DayOnly_WithDaysOfWeek_BadRequest | Validation: DayOnly backfill with DaysOfWeek set returns 400. | PUT backfill DayOnly DaysOfWeek=[Monday] -> Assert HTTP 400 -> "must not be provided for day-only templates". | DayOnly no DaysOfWeek. | Backfill validation DayOnly. |
| 214 | | BackfillTemplatedAvailability_DayOnly_InvalidTimezone_BadRequest | Validation: Invalid timezone returns 400. | PUT backfill DayOnly IanaTimezone="Invalid/Timezone" -> Assert HTTP 400 -> errors["iana_timezone"]. | Timezone validation. | Backfill validation DayOnly. |
| 215 | | BackfillTemplatedAvailability_DayOnly_FromTimeEqualsToTime_BadRequest | Validation: FromTime == ToTime returns 400. | PUT backfill DayOnly FromTime=ToTime -> Assert HTTP 400 -> errors on both fields with "cannot be the same". | Equal time rejection. | Backfill validation DayOnly. |
| 216 | | BackfillTemplatedAvailability_DayOnly_FromTimeAfterToTime_BadRequest | Validation: FromTime > ToTime returns 400. | PUT backfill DayOnly FromTime=17:00, ToTime=09:00 -> Assert HTTP 400 -> "must be before end time". | Time order validation. | Backfill validation DayOnly. |

### Recurring Bad Request Tests

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 217 | | BackfillTemplatedAvailability_Weekly_MissingDaysOfWeek_BadRequest | Validation: Weekly backfill with null DaysOfWeek returns 400. | PUT backfill Weekly DaysOfWeek=null -> Assert HTTP 400 -> "Days of week are required". | DaysOfWeek required. | Backfill validation Weekly. |
| 218 | | BackfillTemplatedAvailability_Weekly_EmptyDaysOfWeek_BadRequest | Validation: Weekly backfill with empty DaysOfWeek returns 400. | PUT backfill Weekly DaysOfWeek=[] -> Assert HTTP 400 -> "Days of week are required". | Empty DaysOfWeek rejected. | Backfill validation Weekly. |
| 219 | | BackfillTemplatedAvailability_Weekly_EndDateBeforeStartDate_BadRequest | Validation: Weekly backfill EndDate < StartDate returns 400. | PUT backfill Weekly EndDate < StartDate -> Assert HTTP 400 -> "must be greater than or equal to start date". | Date range validation. | Backfill validation Weekly. |
| 220 | | BackfillTemplatedAvailability_Weekly_InvalidTimezone_BadRequest | Validation: Weekly backfill invalid timezone returns 400. | PUT backfill Weekly IanaTimezone="Invalid/Timezone" -> Assert HTTP 400. | Timezone validation. | Backfill validation Weekly. |
| 221 | | BackfillTemplatedAvailability_Weekly_FromTimeEqualsToTime_BadRequest | Validation: Weekly backfill FromTime == ToTime returns 400. | PUT backfill Weekly FromTime=ToTime -> Assert HTTP 400 -> "cannot be the same". | Equal time rejection. | Backfill validation Weekly. |
| 222 | | BackfillTemplatedAvailability_Weekly_FromTimeAfterToTime_BadRequest | Validation: Weekly backfill FromTime > ToTime returns 400. | PUT backfill Weekly FromTime=17:00, ToTime=09:00 -> Assert HTTP 400 -> "must be before end time". | Time order validation. | Backfill validation Weekly. |
| 223 | | BackfillTemplatedAvailability_EveryOtherWeek_MissingDaysOfWeek_BadRequest | Validation: EveryOtherWeek backfill with null DaysOfWeek returns 400. | PUT backfill EveryOtherWeek DaysOfWeek=null -> Assert HTTP 400 -> "Days of week are required". | DaysOfWeek required for EveryOtherWeek. | Backfill validation EveryOtherWeek. |
| 224 | | BackfillTemplatedAvailability_EveryOtherWeek_EndDateBeforeStartDate_BadRequest | Validation: EveryOtherWeek backfill EndDate < StartDate returns 400. | PUT backfill EveryOtherWeek EndDate < StartDate -> Assert HTTP 400 -> "must be greater than or equal to start date". | Date range validation. | Backfill validation EveryOtherWeek. |

### Partial Failure Tests

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 225 | | BackfillTemplatedAvailability_OneInvalidTemplate_EntireBatchFails | Atomicity: If one template in a batch is invalid, the entire batch fails and no templates are created. | PUT backfill with 1 valid + 1 invalid (FromTime > ToTime) template -> Assert HTTP 400 -> GET valid template by ID -> Assert HTTP 404 (not created). | Batch atomicity - all-or-nothing semantics. | Backfill atomicity. |

---

## tests/ApiTests/Templates/AvailabilityTemplatesSearchApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 226 | | SearchTemplatedAvailability_TemplatesWithinRange_ReturnsOnlyActiveTemplates | Happy path: Search returns only templates active within the date range (day-only within range, recurring overlapping range, indefinite recurring), excludes ended and not-yet-started. | Create 6 templates (3 active in range, 2 ended before, 1 starts after) -> POST search with 15-30 day range -> Assert 3 results -> Assert correct IDs included and excluded. | Range-based search filtering across DayOnly, recurring with end date, and indefinite recurring templates. | Search success. In scope: date range filtering; DayOnly/Weekly/indefinite templates. |
| 227 | | SearchTemplatedAvailability_MultipleProviderLocations_ReturnsTemplatesForAll | Happy path: Search across multiple provider-location combinations returns all matching templates. | Create templates for 2 different provider-location pairs -> POST search with 2 queries -> Assert 2 results. | Multi-query search. | Search success multiple queries. |
| 228 | | SearchTemplatedAvailability_WhenStartDateTimeUtcAfterEndDateTimeUtc_ReturnsBadRequest | Validation: Search with StartDateTimeUtc > EndDateTimeUtc returns 400. | POST search with reversed date range -> Assert HTTP 400 -> errors contain "start_date_time_utc". | Date range order validation. | Search validation. |
| 229 | | SearchTemplatedAvailability_WhenNoTemplatesExist_ReturnsOkWithEmptyList | Edge case: Search with no matching templates returns 200 with empty list. | POST search for non-existent provider/location -> Assert HTTP 200 -> Assert Templates empty. | Empty result handling. | Search edge case. |

---

## tests/ApiTests/TimeSlots/BatchGetTimeSlotsByIdsApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 230 | | BatchGetTimeSlotsByIds_Success | Happy path: Batch get time slots by IDs returns 200 OK. | GET /v1/time-slots/batch?time_slot_ids=timeSlot#34&time_slot_ids=timeSlot#35 with service JWT -> Assert HTTP 200 OK. | Batch get by IDs basic success. | BatchGet success. |
| 231 | | BatchGetTimeSlotsByIds_BadRequestForTooManyItems | Validation: Requesting 51 time slot IDs (over limit of 50) returns 400. | GET /v1/time-slots/batch with 51 IDs -> Assert HTTP 400 BadRequest. | Batch size limit enforcement. | BatchGet validation. |

---

## tests/ApiTests/TimeSlots/DeleteTimeSlotByIdApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 232 | | DeleteTimeSlotById_ReturnsNoContent | Happy path: Delete a time slot by ID returns 204 NoContent. | DELETE /v1/time-slots/timeSlot#54 with service JWT -> Assert HTTP 204 NoContent. | Basic time slot deletion. | Delete success. |

---

## tests/ApiTests/TimeSlots/TimeSlotsForProviderApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 233 | | PutTimeSlotsForProviderDate_ReturnsBadRequestWhenSlotsDontMatchProviderIdInPath | Validation: Time slot ProviderId doesn't match path provider ID returns 400. | PUT /v1/time-slots/provider/pr_yes/{date} with slot ProviderId="pr_no" -> Assert HTTP 400 BadRequest. | Provider ID mismatch validation. | Put validation. |
| 234 | | PutTimeSlotsForProviderDate_ReturnsBadRequestWhenSlotsDontMatchDateInPath | Validation: Time slot StartTime date doesn't match path date returns 400. | PUT /v1/time-slots/provider/{id}/{date} with slot StartTime on date+1 -> Assert HTTP 400 BadRequest. | Date mismatch validation. | Put validation. |
| 235 | | UpdateOrDeleteTimeSlotsForProviderDateAsync | Happy path: Create 2 slots, verify via GET, update to 1 slot, verify updated fields (location, patient type, included/excluded procedures cleared). | PUT 2 slots (7:30 + 11:30) -> Verify not empty via GET -> PUT 1 slot (11:30, new location, Existing patient type, no procedures) -> GET -> Assert 1 slot with updated fields via BeEquivalentTo (excluding TimeSlotId, approximate CreatedUtc). | Full create-update lifecycle with field-level validation. | Put success. In scope: create; update; field changes; slot count changes. |
| 236 | | DeleteTimeSlotsForProviderDate | Happy path: Create slot, verify exists, "delete" by putting empty list, verify gone. | PUT 1 slot -> GET -> Assert not empty -> PUT empty TimeSlots list -> GET -> Assert empty. | Slot deletion via empty PUT. | Delete via PUT success. |
| 237 | | GetTimeSlotsForProvider_Pagination | Happy path: Create 3 slots, paginate with limit=2, verify page 1 has 2 items + NextPageToken, page 2 has 1 item + null NextPageToken. | PUT 3 slots (7:30, 8:30, 9:30) -> GET limit=2 -> Assert 2 slots + NextPageToken not null -> GET page 2 with token -> Assert 1 slot + NextPageToken null. | Pagination with limit and next page token. | Get pagination. |

---

## tests/ApiTests/TimeSlots/TimeSlotsForProviderLocationApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 238 | | GetTimeSlotsForProviderLocation | Happy path: Create slots across 3 dates for same provider+location, query by location date range, verify all 3 returned with correct fields. | PUT slot date1 (7:30) + PUT slot date2 (11:30) + PUT slot date3 (11:30) -> GET /v1/time-slots/provider/{id}/location/{loc}?start_date=date1&end_date=date3 -> Assert 3 slots via BeEquivalentTo (excluding TimeSlotId, approximate CreatedUtc). | Provider-location date range query with multi-date results. | GetByProviderLocation success. |
| 239 | | GetTimeSlotsForProviderLocation_MoreFiltering | Happy path: Create 2 slots with different SourceTypes (CalendarIntegrationApi and Synchronizer), filter by source=CalendarIntegrationApi, verify only 1 returned. | PUT 2 slots (11:30 CalendarIntegrationApi + 15:30 Synchronizer) -> GET with source=CalendarIntegrationApi filter -> Assert 1 slot with CalendarIntegrationApi source. | Source type filtering on provider-location query. | GetByProviderLocation source filtering. |
| 240 | | BatchGetTimeSlotsForProviderLocation | Happy path: Create slots for 2 different locations across 3 dates, batch query with 2 provider-location-date-range pairs, verify all 3 returned. | PUT slot loc1/date1 + PUT slot loc2/date2 + PUT slot loc2/date3 -> POST /v1/time-slots/provider-location/batch with 2 ranges -> Assert 3 results with correct fields via BeEquivalentTo. | Batch provider-location query across multiple locations and date ranges. | BatchGet success. |

---

## tests/ApiTests/TimeSlots/UpdateTimeSlotByIdApiTests.cs

| # | Do we need these? | Test Name | What It Tests | Steps | Summary | Scope |
|---|-------------------|-----------|---------------|-------|---------|-------|
| 241 | | UpdateTimeSlotById_ReturnsNotFoundWhenTimeSlotDoesNotExist | Error handling: Updating a non-existent time slot returns 404. | PUT /v1/time-slots/{randomId} with update body -> Assert HTTP 404 NotFound. | Non-existent slot returns 404. | Update edge case. |
| 242 | | UpdateTimeSlotById_Success | Happy path: Create slot, update its location/startTime/procedures, verify via batch GET. | PUT provider-date to create slot -> Get timeSlotId -> PUT /v1/time-slots/{timeSlotId} with new location, new date/time, new procedures -> GET batch by ID -> Assert updated fields via BeEquivalentTo (excluding TimeSlotId, approximate CreatedUtc). | Full create-update-verify lifecycle for individual slot updates. | Update success. |
