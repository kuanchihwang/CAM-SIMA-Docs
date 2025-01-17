# 10 - Final steps

Open PRs to the relevant repositories.

## atmospheric_physics
Follow the [atmospheric_physics workflow](../atmospheric_physics/development_workflow.md) to open a PR, respond to review requests, and populate the `NamesNotInDictionary.txt` file.

## CAM
Follow the [CAM workflow](https://github.com/ESCOMP/CAM/wiki/CAM-SE-Workflows) to open a PR and respond to review requests. Be sure to include the `ccpp-conversion` label on your PR. Once you are assigned a tag by the gatekeeper, run the tests, update the Changelog, then make the tag and archive the baselines.

## CAM-SIMA
1. Add a new test for your scheme using the before/after snapshots you generated during conversion. You can parallel existing FPHYStest tests and follow the instructions [here](../development/cam-testing.md#adding-a-new-regression-test).
1. Follow the [CAM-SIMA workflow](../development/cam-sima-workflow.md) to open a PR, run the regression tests, and (potentially) make a tag. At the very least, you will have the testing updates above and a new atmopsheric_physics hash (which will point to your fork until the atmospheric_physics PR is merged).

## Final touches
1. Once all PRs are merged and tagged, check off the scheme as "Converted" in the [spreadsheet](https://docs.google.com/spreadsheets/d/1_1TTpnejam5jfrDqAORCCZtfkNhMRcu7cul37YTr_WM/edit?gid=0#gid=0).