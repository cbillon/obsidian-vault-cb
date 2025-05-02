---
link: https://fabianlee.org/2024/06/23/yq-updating-deeply-nested-elements/
site: "Fabian Lee : Software Engineer"
excerpt: "Mike Farah’s yq yaml processor has a a rich set of operators and
  functions for advanced usage.  In this article, I will illustrate how to
  update deeply nested elements in yaml.  This can be done for both known paths
  as well as arbitrarily deep paths. Sample yaml We will use the following yaml
  files to ... yq: updating deeply nested elements"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/deep
  - slurp/element
  - slurp/nested
  - slurp/yaml
  - slurp/yq
slurped: 2025-03-02T09:56
title: "yq: updating deeply nested elements"
---

Mike Farah’s [yq](https://github.com/mikefarah/yq) yaml processor has a a rich set of operators and functions for advanced usage.  In this article, I will illustrate how to update deeply nested elements in yaml.  This can be done for both known paths as well as arbitrarily deep paths.

## Sample yaml

We will use the following yaml files to illustrate finding and updating the deeply nested ‘tags’ element.

cat <<EOF >company-servers.yaml
company:
  regions: [ 'us-east', 'us-central', 'us-west']
  inventory:
    machineA:
      region: us-east
      tags:
      - linux
    machineB:
      region: us-central
      tags:
      - windows
EOF

## Updating nested yaml element with known path

Updating an element like ‘regions’ would be relatively simple because the dotted path is well known as ‘.company.regions’

$ yq '.company.regions = ["americas","europe"]' company-servers.yaml 
company:
  regions: [americas, europe]
  inventory:
    machineA:
      region: us-east
      tags:
        - linux
    machineB:
      region: us-central
      tags:
        - windows

## Updating nested yaml element with deep arbitrary path

However, updating the ‘tags’ element nested deeply in each machine object would be more difficult to express because of the variability of the ‘machine<ID>’ keys.  Luckily we can use the double dot “..” notation to find and update any arbitrarily nested “tags” element.

$ yq '(.. | select(has("tags")).tags) = ["coreos","arm64"]' company-servers.yaml
company:
  regions: ['us-east', 'us-central', 'us-west']
  inventory:
    machineA:
      region: us-east
      tags:
        - coreos
        - arm64
    machineB:
      region: us-central
      tags:
        - coreos
        - arm64

Or if we wanted to append array elements, we could use the “+=” operator

yq '(.. | select(has("tags")).tags) += ["amd64"]' company-servers.yaml

Or if we wanted to update only the machine objects where the region was “us-east”

yq '(.. | select(has("tags") and .region=="us-east").tags) += ["amd64"]' company-servers.yaml

## Github project

The examples shown above can be pulled down from [my github project](https://github.com/fabianlee/blogcode/tree/master/yq/update-deep).

myproject=https://raw.githubusercontent.com/fabianlee/blogcode
wget $myproject/master/yq/update-deep/company-servers.yaml
wget $myproject/master/yq/update-deep/yq-mikefarah-update-deep-element.sh
chmod +x *.sh

# run script
./yq-mikefarah-update-deep-element.sh

REFERENCES

[yq github project](https://github.com/mikefarah/yq)

[yq manual pages](https://mikefarah.gitbook.io/yq/)

[yq show and tell, examples](https://github.com/mikefarah/yq/discussions/categories/show-and-tell)

[stackoverflow, ‘select’ and ‘has’ to update with yq](https://stackoverflow.com/questions/71764331/using-yq-to-increment-value-if-field-exists)

[yq docs, logic without if/then/else](https://mikefarah.gitbook.io/yq/usage/tips-and-tricks#logic-without-if-elif-else)

[stackoverflow, select in place of ‘if’ statement](https://stackoverflow.com/questions/69273004/update-a-specific-object-value-under-yaml-based-on-a-condition-using-yq)

[Martin Heinz, mastering yq](https://towardsdatascience.com/yq-mastering-yaml-processing-in-command-line-e1ff5ebc0823)

[yq docs, multiple merge operators: + d ? n c](https://mikefarah.gitbook.io/yq/operators/multiply-merge)

[yq docs, and or boolean operators](https://mikefarah.gitbook.io/yq/operators/boolean-operators)

[yq, filter from array and append to another file](https://github.com/mikefarah/yq/discussions/712)

NOTES

**Here are the steps for installing the latest yq on Snap enabled systems like Ubuntu/Debian.**

sudo snap install yq
yq --version

**Manual download and installation of yq**

If you need to install the latest version manually, below are the commands.

# essential packages
sudo apt get install curl jq wget -y
# get latest yq release version
latest_yq_linux=$(curl -sL https://api.github.com/repos/mikefarah/yq/releases/latest | jq -r ".assets[].browser_download_url" | grep linux_amd64.tar.gz)

# download, extract, and put yq binary into PATH
wget $latest_yq_linux
tar xvfz yq_linux_amd64.tar.gz
sudo cp yq_linux_amd64 /usr/local/bin/yq

Example of using ‘jq’ binary

kubectl get ingress $k8s_ingress_name -n $k8s_ingress_ns -o=yaml | yq '.metadata.annotations."ingress.kubernetes.io/backends"' | jq -r 'with_entries(select(.key | startswith("k8s1-"))) | keys[]'