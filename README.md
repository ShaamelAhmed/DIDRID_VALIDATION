# DIDRID_VALIDATION
Standalone black-box validator for Valeo software deliveries. Parses delivery bundles, builds release-specific profiles, runs DID/RID checks through an HTTP bridge and Vehicle Spy backend, and outputs PASS/FAIL results with JSON/CSV evidence logs.
# Valeo Delivery Validator

Simplified multi-release DID/RID black-box validator for Valeo deliveries.

## PLEASE REFER TO DOCUMENT TO EXECUTE< BUILD< BRIDGE AND RUN VALIDATION COMMANDS>>
 
## What it does
1. Ingests three Valeo bundles (Binary, ECU Info, 3PD Report)
2. Extracts release metadata and expected values
3. Builds a release-specific validation profile
4. Runs a fixed DID/RID test set through a local HTTP bridge
5. Writes JSON + CSV results

## Project flow
`deliveries -> ingest_delivery.py -> build_profile.py -> local_bridge.py -> validate_release.py -> outputs`

## Recommended VS Code flow
1. Open the `valeo_delivery_validator` folder in VS Code.
2. Open a terminal in the project root.
3. Run the bridge in terminal 1:
   `python bridge/local_bridge.py`
4. In terminal 2 run:
   `python scripts/ingest_delivery.py --binary-bundle <zip> --ecu-info-bundle <zip> --report-3pd-bundle <zip> --output-dir deliveries/parsed/sample`
5. Build the profile:
   `python scripts/build_profile.py --summary deliveries/parsed/sample/delivery_summary.json --output profiles/generated/sample_profile.json`
6. Validate:
   `python scripts/validate_release.py --profile profiles/generated/sample_profile.json --output-dir outputs/sample_run`

## Current state
- The validator path is fully runnable.
- **F188** (software version) uses live `intrepid.VSpyTextApi` (`send_tx` → `Read F188 DID` on `127.0.0.1:7000`).
- F100 and all RID entries still use mock responses.
- Integration point: `bridge/wrappers/vspy_operation_map.json` + `bridge/wrappers/vspy_did_rid_runner.py`.

## Vehicle Spy (F188)
Uses documented `intrepid.VSpyTextApi` (`send_tx` / `trigger_fb` / `send` + `read_message`).

Prerequisites:
1. Vehicle Spy is already running with TCP/IP Text API enabled (default `127.0.0.1:7000`).
2. The loaded VSpy database/package contains the exact TX / Function Block / message descriptions referenced in `vspy_operation_map.json`.
3. Python can `from intrepid import VSpyTextApi`.

Current F188 mapping (`send_tx`):
- `tx_name`: `Read F188 DID`
- `msg_descriptions`: `["F188 Response"]`

Confirm with:
`python bridge/wrappers/vspy_did_rid_runner.py did F188 --config bridge/wrappers/vspy_operation_map.json`

To temporarily force mock for F188, set `"transport": "mock"` and a `mock_response` in the operation map.
