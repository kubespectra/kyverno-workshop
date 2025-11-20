# Kyverno Deep Dive

The livedemo from the presentation can be found in _Livedemo.md_. The english version of the slides is in _containerdays_kyverno_kubespectra.pdf_, the german version is in _clc_kyverno_kubespectra.pdf_.

The used values for the Kyverno helm chart are in *kyverno-values.yaml*, the used values for the policy reporter are in *reporter-values.yaml*. In *configmap.yaml* is the projectid-mapping used in *policies/validate-namespace.yaml*, in *namespace-filters.yaml* is the configMap used in *policies/validate-image-tag.yaml*