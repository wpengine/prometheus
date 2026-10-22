## WP Engine fork notes

> This is WP Engine's copy of [GoogleCloudPlatform/prometheus](https://github.com/GoogleCloudPlatform/prometheus)
> (Google's GMP fork of Prometheus 2.53.5). It is **not** deployed as a Prometheus server. It exists only as a Go
> module that our services pull in through a `replace` directive. Owned by the **sk8rs** team.

### Why this fork exists

1. Our operators (`web-environment-operator`, and through it `fluid-control-plane-operator`, `fluidctl` and others)
   import `github.com/GoogleCloudPlatform/prometheus-engine/pkg/operator/apis/monitoring/v1` to create
   `PodMonitoring` resources.
2. That package imports `github.com/prometheus/prometheus/google/export` and `.../google/secrets`. Those packages
   exist only in Google's GMP fork, not in upstream Prometheus, so every consumer needs a `replace` such as:

   ```
   replace github.com/prometheus/prometheus => github.com/wpengine/prometheus <pseudo-version>
   ```

   Go ignores `replace` directives in dependencies, so each consumer has to repeat it.
3. Google's `release-2.53.5-gmp` branch does not compile against the `k8s.io/client-go` version our operators use
   (for example `cache.DefaultWatchErrorHandler` gained a `context.Context` argument). This fork carries the
   `client-go` and `klog` fixes needed for it to build, plus our own dependency security updates.

### What differs from Google's `release-2.53.5-gmp`

- `k8s.io/{api,apimachinery,client-go}` updated to v0.36.1 and the call sites in `discovery/kubernetes` fixed.
- `klog/v2` replaced by a `wpengine/klog-gokit` build compatible with the newer `client-go`.
- Go and npm dependency updates to address Dependabot alerts.

This repo does not build on its own (the `klog-gokit` replace is incomplete for `k8s.io/apimachinery`). Verify changes
by building a consumer, such as `web-environment-operator`, with its `replace` pointed at this branch.

### Updating a consumer

1. Merge the change here and note the new commit SHA on the default branch.
2. In each consumer, run `go get github.com/prometheus/prometheus@<sha>` or edit the `replace` line to the new
   pseudo-version (`v0.0.0-<UTC commit timestamp>-<12 character sha>`), then `go mod tidy` and build.

### Direction: move to upstream Prometheus

This fork is a stopgap. Our direction is to use upstream [prometheus/prometheus](https://github.com/prometheus/prometheus),
the open-source standard, in the autoscale platform, and retire this fork.

Upstream does not contain the `google/export` and `google/secrets` packages that `prometheus-engine`'s `monitoring/v1`
imports, so the fork can only go away once the consumers no longer depend on that GMP-specific package. Until then,
keep this fork building against the `client-go` version the consumers use, and do not add new consumers of it.

---

## Google Cloud Managed Service for Prometheus (GMP) Fork

> NOTICE: This repository is a fork of [github.com/prometheus/prometheus](https://github.com/prometheus/prometheus) that includes support for GMP.
>
> We actively work on ensuring vanilla Prometheus can work with GMP; this fork will
> be significantly reduced. Notably, the custom GCM export will be removed.

For GMP specific documentation and to get started, go to [g.co/cloud/managedprometheus](https://g.co/cloud/managedprometheus).

Otherwise, refer to https://github.com/prometheus/prometheus documentation.
