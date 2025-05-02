---
link: https://fabianlee.org/2024/06/22/yq-validate-yaml-syntax/
site: "Fabian Lee : Software Engineer"
excerpt: 'Mike Farah’s yq yaml processor has a a full-featured validation command that is very detailed in its reporting, but the yaml specification itself is very lenient, which means yq may accept scenarios you did not expect (e.g. an empty file). yq -v file.yaml >/dev/null ; echo "final result = $?" Luckily, the yq tips-and-tricks section ... yq: validate yaml syntax'
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/yaml
  - slurp/yq
slurped: 2025-03-02T09:54
title: "yq: validate yaml syntax"
"tags:":
---

June 22, 2024

  

Categories: [Linux](https://fabianlee.org/category/linux/)  

Mike Farah’s [yq](https://github.com/mikefarah/yq) yaml processor has a a full-featured validation command that is very detailed in its reporting, but the [yaml specification](https://yaml.org/spec/1.2.2/) itself is very lenient, which means yq may accept scenarios you did not expect (e.g. an empty file).

yq -v file.yaml >/dev/null ; echo "final result = $?"

Luckily, the [yq tips-and-tricks section](https://mikefarah.gitbook.io/yq/usage/tips-and-tricks#validating-yaml-files) provides an example of testing whether the yaml file is of valid syntax AND either a map or an array at the top level, which aligns better with what most would consider a valid yaml file.

yq --exit-status 'tag == "!!map" or tag== "!!seq"' file.yaml > /dev/null ; echo "final result = $?"

REFERENCES

[Mike Farah yq on Github](https://github.com/mikefarah/yq)

[stackoverflow, how to validate yaml using yq](https://stackoverflow.com/questions/75920947/how-to-validate-yaml-using-yq)

[YAML 1.2 specification](https://yaml.org/spec/1.2.2/)