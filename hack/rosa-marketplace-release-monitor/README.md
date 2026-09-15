# ROSA Marketplace release monitor verification

The fixture test harness can run the checks for each implementation step
independently:

```console
bash hack/rosa-marketplace-release-monitor-tests.sh release-controller-api
bash hack/rosa-marketplace-release-monitor-tests.sh payload-selection
bash hack/rosa-marketplace-release-monitor-tests.sh rhcos-extraction
bash hack/rosa-marketplace-release-monitor-tests.sh prow-contract
bash hack/rosa-marketplace-release-monitor-tests.sh publisher
```

The release-controller groups verify the updated API sequence explicitly:

1. validate the target stream through `/config`;
2. query `/api/v1/releasestreams/ready` and select the earliest ready tag;
3. resolve that exact tag through `/tags`;
4. extract and validate the installer-provided RHCOS metadata.

The tests also verify that an empty ready response stops before the tags call,
and that a selected ready tag may transition to `Accepted` or `Rejected` while
it is being resolved.

Run the complete fixture and configuration-contract suite before requesting a
Prow rehearsal:

```console
bash hack/rosa-marketplace-release-monitor-tests.sh all
```

After regenerating the registry metadata and Prow jobs, run the repository
configuration validators locally:

```console
make registry-metadata
make jobs
make ci-operator-checkconfig
make validate-step-registry
make checkconfig
git diff --check
```

On the `openshift/release` pull request, rehearse the full verification
presubmit and the generated periodic with:

```text
/pj-rehearse pull-ci-openshift-release-main-rosa-marketplace-release-monitor-test periodic-ci-openshift-release-main-rosa-marketplace-release-rosa-marketplace-release-detect
```

The rehearsal must pass while `MARKETPLACE_PUBLISH_ENABLED=false`. Review the
detector state and uploaded artifacts, then acknowledge the result with
`/pj-rehearse ack`.
