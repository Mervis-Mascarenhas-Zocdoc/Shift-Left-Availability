# interop-availability-api Test Mapping — Changelog

## 2026-07-23 — initial mapping @ 2a20858

Initial mapping of `tests/IntegrationTests` (test-type: general, branch `main`).

- 14 test files
- 102 test cases (including parametrized cases expanded per case)

### Files
- tests/IntegrationTests/AvailabilityTemplate/AvailabilityTemplatePersistenceTests.cs — 16 tests
- tests/IntegrationTests/ExclusionPersistenceTests.cs — 33 tests (incl. 16 `GetExclusionsByEntityIdsAndRangeV2` cases + 4 `BatchUpdateExclusions` cases)
- tests/IntegrationTests/Lambdas/AvailabilityDataHandlerIntegrationTests.cs — 1 test
- tests/IntegrationTests/Lambdas/DocumentServiceSubscriberIntegrationTests.cs — 2 tests
- tests/IntegrationTests/Lambdas/ExclusionDdbEventStreamHandlerTests.cs — 9 tests
- tests/IntegrationTests/Lambdas/ExclusionExpirationIntegrationTests.cs — 1 test
- tests/IntegrationTests/Lambdas/LeadTimeDataHandlerIntegrationTests.cs — 3 tests
- tests/IntegrationTests/Lambdas/PracticeLeadTimeRewardUpdateHandlerTests.cs — 4 tests (incl. 3 `ProcessSqsMessages_CalculatesRewardEligibility` cases)
- tests/IntegrationTests/Lambdas/TriggerLeadTimeRewardUpdateHandlerTests.cs — 1 test
- tests/IntegrationTests/LeadTime/BusinessHoursPersistenceTests.cs — 6 tests
- tests/IntegrationTests/LeadTime/LeadTimePersistenceTests.cs — 6 tests
- tests/IntegrationTests/LeadTime/LeadTimePracticeConfigPersistenceTests.cs — 3 tests
- tests/IntegrationTests/TimeSlots/TimeSlotPersistenceTests.cs — 14 tests
- tests/IntegrationTests/TimeSlots/TimeSlotServiceIntegrationTests.cs — 3 tests (incl. 2 `UpdateOrDeleteTimeSlotsForProviderDateAsync` cases)
