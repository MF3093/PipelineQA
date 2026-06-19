# Assumptions, Questions & Discrepancies Log — TEST-HARNESS-NO-EPICS

| ID | Story Key | Status | Description | Resolution |
|----|-----------|--------|-------------|------------|
| Q-001 | TEST-SAA-008 | Open | No validation rules defined for modal fields (Item Name, Quantity, Unit Price, Category) — required-field enforcement, format constraints, min/max values, and corresponding error messages are all unspecified. Affects test case design for negative/edge-case scenarios. | |
| Q-002 | TEST-SAA-008 | Open | AC-3 describes only the success path for Save (row updates immediately). Failure path is undefined — what happens on network error or server-side save failure? No error message or recovery behavior specified. | |
| Q-003 | TEST-SAA-008 | Open | The story does not explicitly state whether the quick edit modal opens pre-populated with the item's existing data. This is critical to verify: testing empty-modal vs. pre-populated-modal are entirely different scenarios. | |
| Q-004 | TEST-SAA-008 | Open | Category field type is not specified (free-text input vs. dropdown/select). The available options and whether the field can be cleared are unknown. This affects how the field is tested and what valid/invalid inputs apply. | |
| Q-005 | TEST-SAA-008 | Open | AC-4 states Cancel closes the modal with no changes saved, but does not clarify whether a confirmation prompt appears if the user has made unsaved edits. Immediate close vs. "discard changes?" dialog is a testable behavioral difference. | |
