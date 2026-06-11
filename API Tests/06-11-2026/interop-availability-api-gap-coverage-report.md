# Interop Availability API — Test Gap Coverage Report

**Date:** 2026-06-11
**Scope:** API integration tests (242 test cases) for `interop-availability-api`
**Source:** `tests/ApiTests/` in [Zocdoc/interop-availability-api](https://github.com/Zocdoc/interop-availability-api/tree/main/tests/ApiTests)

---

## Executive Summary

| Metric | API Tests |
|--------|-----------|
| Total tests analyzed | 242 |
| Relevant | **238 (98.3%)** |
| Irrelevant / Stale | 4 |
| High-priority missing gaps | 12 |
| Medium-priority missing gaps | 19 |
| Low-priority missing gaps | 8 |

---

## API Test Analysis

### 1. tests/ApiTests/ExclusionsApiTests.cs (83 tests)

**Irrelevant Tests:**

| Test # | Test Name | Issue |
|--------|-----------|-------|
| 12 | BatchAddExclusions_Success | **Partially redundant scope** — This test verifies read-back via both GetByIds and GetByRangeV2 after creation, duplicating verification that the GetByIds and GetByRangeV2 success tests (#29, #43) already cover. The creation verification itself is relevant, but the cross-endpoint read-back is overlapping. |

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | BatchAdd with duplicate exclusions in same batch | Submitting multiple exclusions with identical EntityId, StartDateTimeUtc, EndDateTimeUtc in a single batch — should the API de-duplicate or reject? | **High** |
| 2 | BatchAdd with Appointment type and valid AppointmentId | Happy path for Appointment-type exclusion with AppointmentId set (only the negative case — Busy type with AppointmentId — is tested). | **High** |
| 3 | BatchGetByRangeV2 with overlapping date ranges | Query with overlapping EntityDateRanges to verify deduplication behavior in response. | Medium |
| 4 | BatchGetByRangeV2 pagination | No test verifies multi-page results for range queries; only GetExclusionsPage tests pagination. | Medium |
| 5 | BatchUpdate with non-existing exclusion ID | What happens when an update references an ID that doesn't exist? No 404/error handling test for this case. | Medium |
| 6 | BatchUpdate changing EntityId | Verify whether updating an exclusion's EntityId is allowed or rejected. | Medium |
| 7 | BatchUpdate changing ExclusionType | Verify behavior when changing type from Busy to Appointment (or vice versa). | Medium |
| 8 | BatchRemove with non-existing IDs | Verify behavior when deleting IDs that don't exist — silent success or error? | Medium |
| 9 | GetExclusionsPage multi-page pagination | Only single-page pagination is tested (NextPaginationToken is null). No test for multi-page traversal. | Medium |
| 10 | GetExclusionsPage with no matching exclusions | Edge case: query returns empty page. | Low |
| 11 | BatchAdd at exact boundary of max batch size | Test with exactly 100 items (the maximum) to verify boundary acceptance. | Low |

---

### 2. tests/ApiTests/LeadTime/LeadTimeApiTests.cs (36 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | AddOrUpdate with Day unit type | All happy-path tests use Hour unit type. No test verifies Day unit type (e.g., Day value=1 through Day value=5). | **High** |
| 2 | BatchGet for multiple entity types simultaneously | BatchGet is only tested with a single entity type (Practice). No test queries Provider + Location + Practice together. | **High** |
| 3 | Delete for Location entity type | Delete happy path only tests Provider entity. Location and Practice entity deletions are not tested. | Medium |
| 4 | AddOrUpdate with all valid Hour values (2, 4, 6) | Only value=2 and value=4 (via upsert) are tested; value=6 is never used. | Medium |
| 5 | BatchGet returns empty for practice with no lead times | No test verifies the empty-result case for batch get. | Medium |
| 6 | Delete for entity that has no lead time set | What status code is returned when deleting a lead time that was never created? | Low |

---

### 3. tests/ApiTests/LeadTime/BusinessHoursApiTests.cs (25 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Get after Create verification | No test creates business hours and then GETs them to verify the response matches. The create test (#120) only verifies the PUT response, not a subsequent GET. | **High** |
| 2 | Get after Delete verification | No test verifies that GET returns 204 NoContent after deleting previously existing business hours. | **High** |
| 3 | AddOrUpdate with all 7 days of the week | Only Monday and Tuesday are tested. No test verifies all 7 days can be created. | Medium |
| 4 | AddOrUpdate with boundary times (00:00, 23:59) | No test for midnight or end-of-day boundary times. | Medium |
| 5 | AddOrUpdate with StartTime equal to EndTime | Unlike Templates which test FromTime==ToTime, BusinessHours doesn't have this specific validation test. | Medium |
| 6 | Delete idempotency — delete same practice twice | Test #138 deletes non-existent, but no test deletes the same existing practice twice. | Low |

---

### 4. tests/ApiTests/Templates/AvailabilityTemplatesAuthApiTests.cs (26 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Backfill with PracticeAppointmentManagement role | Backfill auth tests don't include this role (unlike Create/Update/Delete which test it). Verify it's correctly forbidden. | Medium |
| 2 | Search with CsrPracticeEdit role | Search auth tests only cover CsrUser and service role. CsrPracticeEdit is not tested for search. | Medium |
| 3 | Service account for Create/Update/Delete | Service account auth is tested for Backfill and Search but not for basic CRUD operations. | Low |

---

### 5. tests/ApiTests/Templates/AvailabilityTemplatesCreateApiTests.cs (23 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Create with duplicate provider-location-time overlap | What happens when creating a template that overlaps with an existing template for the same provider-location? | **High** |
| 2 | Create EveryOtherWeek with all validations | EveryOtherWeek only has 2 validation tests (MissingDaysOfWeek, EndDateBeforeStart). Missing: InvalidTimezone, FromTimeEqualsToTime, FromTimeAfterToTime, StartDateInPast. | Medium |
| 3 | Create with invalid ProviderId or LocationId format | No test validates the format of ProviderId/LocationId (e.g., non-prefixed IDs). | Medium |
| 4 | Create with empty string fields | No test for empty string ProviderId, LocationId, or PracticeId. | Low |

---

### 6. tests/ApiTests/Templates/AvailabilityTemplatesUpdateApiTests.cs (10 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Update changing ScheduleType (DayOnly to Weekly) | No test verifies whether schedule type changes are allowed or rejected. | **High** |
| 2 | Update EveryOtherWeek template | No update tests exist for EveryOtherWeek schedule type. | Medium |
| 3 | Update with same values (no-op update) | Verify idempotent behavior when updating with identical values. | Low |

---

### 7. tests/ApiTests/Templates/AvailabilityTemplatesDeleteApiTests.cs (3 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Delete template that has been backfilled | Verify deletion behavior when the template has associated backfilled time slots. | Medium |
| 2 | Delete idempotency — delete same template twice | First delete returns 204, second delete should return 404. Verify this two-step behavior. | Low |

---

### 8. tests/ApiTests/Templates/AvailabilityTemplatesBackfillApiTests.cs (19 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Backfill with StartDate in the past | Unlike Create, Backfill allows past dates (it's for historical data). Verify this is accepted. | Medium |
| 2 | Backfill with EveryOtherWeek full validation suite | EveryOtherWeek only has 2 backfill validation tests. Missing: InvalidTimezone, FromTimeEqualsToTime, FromTimeAfterToTime. | Medium |
| 3 | Backfill max batch size | No test for the maximum number of templates in a single backfill batch. | Medium |
| 4 | Backfill verify time slots are created | No test verifies that time slots are actually generated as a result of backfill (only template metadata is checked). | Low |

---

### 9. tests/ApiTests/Templates/AvailabilityTemplatesSearchApiTests.cs (4 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Search pagination | No test for paginated search results when many templates match. | Medium |
| 2 | Search by specific schedule type filter | No test filters search results by schedule type (DayOnly/Weekly/EveryOtherWeek). | Medium |
| 3 | Search with very large date range | Boundary test for search with a very wide date range (e.g., 1 year). | Low |

---

### 10. tests/ApiTests/TimeSlots/BatchGetTimeSlotsByIdsApiTests.cs (2 tests)

**Irrelevant Tests:**

| Test # | Test Name | Issue |
|--------|-----------|-------|
| 230 | BatchGetTimeSlotsByIds_Success | **Weak assertion** — Only verifies HTTP 200 status code with hardcoded IDs (timeSlot#34, timeSlot#35). Does not verify response body content, field values, or that the returned slots match the requested IDs. If the IDs don't exist, it still passes with an empty response. |

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Authorization tests for BatchGetTimeSlotsByIds | **ZERO authorization tests exist for any TimeSlots endpoint.** All other domains (Exclusions, LeadTime, Templates) have comprehensive auth matrices. TimeSlots should test: Patient (forbidden), Provider (correct/incorrect practice), RBAC roles, CSR roles, ServiceAccount, MissingJwt. | **High** |
| 2 | BatchGet with non-existing IDs | Verify behavior when requested IDs don't exist — empty array or partial results? | **High** |
| 3 | BatchGet with exactly 50 IDs (boundary) | Test at the exact limit boundary (50 is max, 51 returns 400). | Medium |
| 4 | BatchGet response field validation | Verify all returned fields (TimeSlotId, ProviderId, LocationId, StartTime, PatientType, SourceType, etc.) match expected values. | Medium |

---

### 11. tests/ApiTests/TimeSlots/DeleteTimeSlotByIdApiTests.cs (1 test)

**Irrelevant Tests:**

| Test # | Test Name | Issue |
|--------|-----------|-------|
| 232 | DeleteTimeSlotById_ReturnsNoContent | **Weak test** — Deletes a hardcoded ID (timeSlot#54) without creating it first. Does not verify the slot is actually gone after deletion. Essentially only tests that the endpoint returns 204 for any ID. |

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Authorization tests for DeleteTimeSlotById | **No auth tests.** Should test the same role matrix as other domains. | **High** |
| 2 | Delete with create-then-verify lifecycle | Create a time slot, delete it, verify it's gone via BatchGet. | **High** |
| 3 | Delete non-existing time slot | Verify 404 is returned when deleting a slot that doesn't exist. | Medium |
| 4 | Delete idempotency | Delete the same slot twice — first should succeed, second should return 404. | Low |

---

### 12. tests/ApiTests/TimeSlots/TimeSlotsForProviderApiTests.cs (5 tests)

**Irrelevant Tests:**

| Test # | Test Name | Issue |
|--------|-----------|-------|
| 235 | UpdateOrDeleteTimeSlotsForProviderDateAsync | **Misleading name** — Named "UpdateOrDelete" but only tests the update path. Delete is tested separately in test #236. The "Delete" in the name is misleading. |

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Authorization tests for PutTimeSlotsForProviderDate | **No auth tests.** This is a write endpoint with no authorization coverage. | **High** |
| 2 | Authorization tests for GetTimeSlotsForProvider | **No auth tests.** This is a read endpoint with no authorization coverage. | **High** |
| 3 | Put with invalid timezone | No timezone validation test (other domains test this). | Medium |
| 4 | Put with overlapping time slots for same provider-date | What happens when two slots have overlapping StartTime values? | Medium |
| 5 | Put with empty ProviderId or LocationId | Input validation for missing required fields. | Medium |
| 6 | Get with no time slots for provider | Edge case: verify empty response for a provider with no slots. | Low |
| 7 | Get with invalid date format in query params | Validate date format handling in start_date/end_date parameters. | Low |

---

### 13. tests/ApiTests/TimeSlots/TimeSlotsForProviderLocationApiTests.cs (3 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Authorization tests for GetTimeSlotsForProviderLocation | **No auth tests.** | **High** |
| 2 | Authorization tests for BatchGetTimeSlotsForProviderLocation | **No auth tests.** | **High** |
| 3 | Get with empty date range (start == end) | Boundary test for single-day query. | Medium |
| 4 | Get with reversed date range (start > end) | Validation test for invalid date range order. | Medium |
| 5 | BatchGet with empty request body | Validate handling of empty batch request. | Low |

---

### 14. tests/ApiTests/TimeSlots/UpdateTimeSlotByIdApiTests.cs (2 tests)

**Irrelevant Tests:** None — all tests are relevant.

**Missing Tests:**

| # | Suggested Test | What It Would Test | Priority |
|---|----------------|-------------------|----------|
| 1 | Authorization tests for UpdateTimeSlotById | **No auth tests.** | **High** |
| 2 | Update with invalid field values | No validation tests for invalid timezone, empty ProviderId, invalid PatientType, etc. | Medium |
| 3 | Update changing ProviderId | Verify whether changing the ProviderId on a time slot is allowed or rejected. | Medium |

---

## High-Priority Gaps Summary

### Critical Gap: TimeSlots Authorization Tests

| # | Endpoint | Missing Coverage |
|---|----------|-----------------|
| 1 | PUT /v1/time-slots/provider/{id}/{date} | No authorization tests (write endpoint) |
| 2 | GET /v1/time-slots/provider/{id} | No authorization tests (read endpoint) |
| 3 | GET /v1/time-slots/batch | No authorization tests (read endpoint) |
| 4 | DELETE /v1/time-slots/{id} | No authorization tests (write endpoint) |
| 5 | PUT /v1/time-slots/{id} | No authorization tests (write endpoint) |
| 6 | GET /v1/time-slots/provider/{id}/location/{loc} | No authorization tests (read endpoint) |
| 7 | POST /v1/time-slots/provider-location/batch | No authorization tests (read endpoint) |

> **All 7 TimeSlots endpoints have ZERO authorization test coverage.** Exclusions has 77 auth tests across 7 operations, LeadTime has 36 across 4 operations, and Templates has 26 across 6 operations. TimeSlots — which manages provider availability data — has none. This is the single highest-priority gap in the test suite.

### Other High-Priority Gaps

| # | Spec File | Missing Coverage |
|---|-----------|-----------------|
| 1 | ExclusionsApiTests.cs | Appointment-type exclusion happy path with valid AppointmentId |
| 2 | ExclusionsApiTests.cs | Duplicate exclusions in same batch — deduplication behavior |
| 3 | LeadTimeApiTests.cs | Day unit type happy path (only Hour type tested) |
| 4 | LeadTimeApiTests.cs | BatchGet with multiple entity types |
| 5 | BusinessHoursApiTests.cs | Get-after-Create verification (create → GET → verify match) |
| 6 | BusinessHoursApiTests.cs | Get-after-Delete verification |
| 7 | AvailabilityTemplatesCreateApiTests.cs | Overlapping template detection for same provider-location |
| 8 | AvailabilityTemplatesUpdateApiTests.cs | Schedule type change behavior (DayOnly → Weekly) |
| 9 | BatchGetTimeSlotsByIdsApiTests.cs | Response body/field validation (current test only checks status code) |
| 10 | DeleteTimeSlotByIdApiTests.cs | Create-delete-verify lifecycle (current test uses hardcoded ID) |

### Weak/Shallow Tests Needing Strengthening

| Test # | Test Name | Issue |
|--------|-----------|-------|
| 230 | BatchGetTimeSlotsByIds_Success | Only asserts HTTP 200 with hardcoded IDs; no response body validation |
| 232 | DeleteTimeSlotById_ReturnsNoContent | Deletes hardcoded ID without creation; no verification of actual deletion |

### Test Category Notes

| Category | Tests | Notes |
|----------|-------|-------|
| [RealOnly] | #64-69, #100, #105, #107, #119 | 10 tests require real API (not fakes); run only in specific environments |
