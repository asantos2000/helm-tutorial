# Getting Started with a Chart Template

## A FIRST TEMPLATE
```bash
# Create chart
helm create mychart
# Output
Creating mychart
##

# Clean-up boilerplate
rm -rf mychart/templates/*.*
rm -rf mychart/templates/tests

# Go to mychart dir
cd mychart/

# Create firts template
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
EOF

# Install
helm install .
# Output
NAME: opulent-ibex
LAST DEPLOYED: Tue Jan  6 13:10:05 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                DATA      AGE
mychart-configmap   1         5m
##

# Get manifest
helm get manifest opulent-ibex
# Output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
##

# Delete
helm delete opulent-ibex
```

## Adding a Simple Template Call
```bash
# Change name
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
EOF

# Testing
helm install --debug --dry-run .
# Output
[debug] Created tunnel using local port: '46263'

[debug] SERVER: "127.0.0.1:46263"

[debug] Original chart version: ""
[debug] CHART PATH: /home/anderson/dev/helm-tutorial/mychart

NAME:   toned-ladybug
REVISION: 1
RELEASED: Sun Jan  6 13:31:09 2019
CHART: mychart-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
fullnameOverride: ""
image:
  pullPolicy: IfNotPresent
  repository: nginx
  tag: stable
ingress:
  annotations: {}
  enabled: false
  hosts:
  - chart-example.local
  paths: []
  tls: []
nameOverride: ""
nodeSelector: {}
replicaCount: 1
resources: {}
service:
  port: 80
  type: ClusterIP
tolerations: []

HOOKS:
MANIFEST:

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: toned-ladybug-configmap
data:
  myvalue: "Hello World"
##
```

## Built-in Objects
* `Release`: This object describes the release itself. It has several objects inside of it:
    * `Release.Name`: The release name.
    * `Release.Time`: The time of the release
    * `Release.Namespace`: The namespace to be released into (if the manifest doesn’t override)
    * `Release.Service`: The name of the releasing service (always Tiller).
    * `Release.Revision`: The revision number of this release. It begins at 1 and is incremented for each helm upgrade.
    * `Release.IsUpgrade`: This is set to true if the current operation is an upgrade or rollback.
    * `Release.IsInstall`: This is set to true if the current operation is an install.
* `Values`: Values passed into the template from the values.yaml file and from user-supplied files. By default, Values is empty.
* `Chart`: The contents of the `Chart.yaml` file. Any data in `Chart.yaml` will be accessible here. For example `{{.Chart.Name}}-{{.Chart.Version}}` will print out the `mychart-0.1.0`.
* The available fields are listed in the [Charts Guide](https://github.com/helm/helm/blob/master/docs/charts.md#the-chartyaml-file)
* `Files`: This provides access to all non-special files in a chart. While you cannot use it to access templates, you can use it to access other files in the chart. See the section Accessing Files for more.
    * `Files.Get` is a function for getting a file by name (`.Files.Get config.ini`)
    * `Files.GetBytes` is a function for getting the contents of a file as an array of bytes instead of as a string. This is useful for things like images.
* `Capabilities`: This provides information about what capabilities the Kubernetes cluster supports.
    * `Capabilities.APIVersions` is a set of versions.
    * `Capabilities.APIVersions.Has $version` indicates whether a version (batch/v1) is enabled on the cluster.
    * `Capabilities.KubeVersion` provides a way to look up the Kubernetes version. It has the following values: `Major`, `Minor`, `GitVersion`, `GitCommit`, `GitTreeState`, `BuildDate`, `GoVersion`, `Compiler`, and `Platform`.
    * `Capabilities.TillerVersion` provides a way to look up the Tiller version. It has the following values: SemVer, GitCommit, and GitTreeState.
* `Template`: Contains information about the current template that is being executed
    * `Name`: A namespaced filepath to the current template (e.g. `mychart/templates/mytemplate.yaml`)
    * `BasePath`: The namespaced path to the templates directory of the current chart (e.g. `mychart/templates`).

## Values Files

`.Value` content is:
1.  The `values.yaml` file in the chart
2.  If this is a subchart, the `values.yaml` file of a parent chart
3.  A values file is passed into `helm install` or `helm upgrade` with the `-f` flag (`helm install -f myvals.yaml ./mychart`)
4.  Individual parameters passed with `--set` (such as `helm install --set foo=bar ./mychart`)

```bash
# Create value file
cat > values.yaml << EOF
favoriteDrink: coffee
EOF

# Modify template
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favoriteDrink }}
EOF

# Test it
helm install . --dry-run --debug
# Output
[debug] Created tunnel using local port: '39259'

[debug] SERVER: "127.0.0.1:39259"

[debug] Original chart version: ""
[debug] CHART PATH: /home/anderson/dev/helm-tutorial/mychart

NAME:   lunging-rattlesnake
REVISION: 1
RELEASED: Sun Jan  6 14:02:09 2019
CHART: mychart-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
favoriteDrink: coffee

HOOKS:
MANIFEST:

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: lunging-rattlesnake-configmap
data:
  myvalue: "Hello World"
  drink: coffee
##

# Passing values in commnad line
helm install --dry-run --debug --set favoriteDrink=slurm .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vociferous-poodle-configmap
data:
  myvalue: "Hello World"
  drink: slurm
##

# More values
cat > values.yaml << EOF
favorite:
  drink: coffee
  food: pizza
EOF

# Modify template for new values
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink }}
  food: {{ .Values.favorite.food }}
EOF
```
## DELETING A DEFAULT KEY

```bash
helm install --dry-run --debug --set favorite.drink=slurm --set favorite.food=null .
# Output
[debug] Created tunnel using local port: '34597'

[debug] SERVER: "127.0.0.1:34597"

[debug] Original chart version: ""
[debug] CHART PATH: /home/anderson/dev/helm-tutorial/mychart

NAME:   washed-kitten
REVISION: 1
RELEASED: Sun Jan  6 14:16:16 2019
CHART: mychart-0.1.0
USER-SUPPLIED VALUES:
favorite:
  drink: slurm
  food: null

COMPUTED VALUES:
favorite:
  drink: slurm
  food: null

HOOKS:
MANIFEST:

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: washed-kitten-configmap
data:
  myvalue: "Hello World"
  drink: slurm
  food:
```

## Template Functions and Pipelines

Helm has over 60 available functions. Some of them are defined by the [Go template language](https://godoc.org/text/template) itself. Most of the others are part of the [Sprig template library](https://godoc.org/github.com/Masterminds/sprig). We’ll see many of them as we progress through the examples.

```bash
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ quote .Values.favorite.drink }}
  food: {{ quote .Values.favorite.food }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: modest-llama-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
##
```

## PIPELINES

```bash
# Rewrite using pipeline
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | quote }}
  food: {{ .Values.favorite.food | quote }}
EOF

# And adding more functions
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: viable-jellyfish-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
##

# repeat COUNT STRING
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | repeat 5 | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: austere-giraffe-configmap
data:
  myvalue: "Hello World"
  drink: "coffeecoffeecoffeecoffeecoffee"
  food: "PIZZA"
##
```

## USING THE DEFAULT FUNCTION

```bash
# Rewrite using pipeline
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | quote }}
EOF

# Delete favorite.drink from values.yaml or pass null
helm install --dry-run --debug . --set favorite.drink=null
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: khaki-scorpion-configmap
data:
  myvalue: "Hello World"
  drink: "tea"
  food: "PIZZA"
##
```

## Flow Control

Control structures:
* `if`/`else` for creating conditional blocks
* `with` to specify a scope
* `range`, which provides a “for each”-style loop

### IF/ELSE

A pipeline is evaluated as _false_ if the value is:
* a boolean false
* a numeric zero
* an empty string
* a `nil` (empty or null)
* an empty collection (`map`, `slice`, `tuple`, `dict`, `array`)

```bash
# Using conditionals
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{ if and (.Values.favorite.drink) (eq .Values.favorite.drink "coffee") }}mug: true{{ end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: unhinged-unicorn-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  mug: true
##

helm install --dry-run --debug . --set favorite.drink=slurm
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alternating-spaniel-configmap
data:
  myvalue: "Hello World"
  drink: "slurm"
  food: "PIZZA"
##
```

### CONTROLLING WHITESPACE

```bash
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{- if and (.Values.favorite.drink) (eq .Values.favorite.drink "coffee") }}
  mug: true
  {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: full-hydra-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  mug: true
##
```

### MODIFYING SCOPE USING WITH

```bash
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- if and (.drink) (eq .drink "coffee") }}
  mug: true
  {{- end }}
  {{- end }} # End of scope
  release: {{ .Release.Name }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: full-hydra-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  mug: true
  release: saucy-magpie
##
```

### LOOPING WITH THE RANGE ACTION

```bash
cat > values.yaml << EOF
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
EOF

# Itarate through list and tupple
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}
  sizes: |-
    {{- range tuple "small" "medium" "large" }}
    - {{ . }}
    {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: saucy-gerbil-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  toppings: |-
    - "Mushrooms"
    - "Cheese"
    - "Peppers"
    - "Onions"
  sizes: |-
    - small
    - medium
    - large
```

In addition to lists and tuples, `range` can be used to iterate over collections that have a key and a value (like a `map` or `dict`).

## Variables

```bash
# Set variable relname out of .Values scope and then use it inside.
# In toppings capture both the index and the value
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- \$relname := .Release.Name -}}
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ \$relname }}
  {{- end }}
  toppings: |-
    {{- range \$index, \$topping := .Values.pizzaToppings }}
      {{ \$index }}: {{ \$topping }}
    {{- end }}
  sizes: |-
    {{- range tuple "small" "medium" "large" }}
    - {{ . }}
    {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: luminous-flee-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  release: luminous-flee
  toppings: |-
      0: mushrooms
      1: cheese
      2: peppers
      3: onions
  sizes: |-
    - small
    - medium
    - large
##

# For data structures that have both a key and a value
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- range \$key, \$val := .Values.favorite }}
  {{ \$key }}: {{ \$val | quote }}
  {{- end}}
EOF
```
## Named Templates

### DECLARING AND USING TEMPLATES WITH DEFINE AND TEMPLATE

```bash
cat > templates/configmap.yaml << EOF
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range \$key, \$val := .Values.favorite }}
  {{ \$key }}: {{ \$val | quote }}
  {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: bumptious-cheetah-configmap
  labels:
    generator: helm
    date: 2019-01-06
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
##

# Moving templates to _helpers.tpl
cat > templates/_helpers.tpl << EOF
{{/* Generate basic labels */}}
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}
EOF

cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range \$key, \$val := .Values.favorite }}
  {{ \$key }}: {{ \$val | quote }}
  {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: bumptious-cheetah-configmap
  labels:
    generator: helm
    date: 2019-01-06
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
##
```

### SETTING THE SCOPE OF A TEMPLATE

```bash
# Moving templates to _helpers.tpl
cat > templates/_helpers.tpl << EOF
{{/* Generate basic labels */}}
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
    chart: {{ .Chart.Name }}
    version: {{ .Chart.Version }}
{{- end }}
EOF

# Passing scope to a template
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" . }}
data:
  myvalue: "Hello World"
  {{- range \$key, \$val := .Values.favorite }}
  {{ \$key }}: {{ \$val | quote }}
  {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: jumpy-hedgehog-configmap
  labels:
    generator: helm
    date: 2019-01-06
    chart: mychart
    version: 0.1.0
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
##
```

### THE INCLUDE FUNCTION

```bash
# Moving templates to _helpers.tpl
cat > templates/_helpers.tpl << EOF
{{- define "mychart.app" -}}
app_name: {{ .Chart.Name }}
app_version: "{{ .Chart.Version }}+{{ .Release.Time.Seconds }}"
{{- end -}}
EOF

# Using include to import the contents of a template into the present pipeline
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels: 
    {{- include "mychart.app" . | nindent 4 }}
data:
  myvalue: "Hello World"
  {{- range \$key, \$val := .Values.favorite }}
  {{ \$key }}: {{ \$val | quote }}
  {{- end }}
  {{- include "mychart.app" . | nindent 2 }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: luminous-manta-configmap
  labels:
    app_name: mychart
    app_version: "0.1.0+1546807133"
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
  app_name: mychart
  app_version: "0.1.0+1546807133"
##
```

> It is considered preferable to use include over template in Helm templates simply so that the output formatting can be handled better for YAML documents.

## Accessing Files Inside Templates

How this works:

* It is okay to add extra files to your Helm chart. These files will be bundled and sent to Tiller. Be careful, though. Charts must be smaller than 1M because of the storage limitations of Kubernetes objects.
* Some files cannot be accessed through the `.Files` object, usually for security reasons.
    * Files in `templates/` cannot be accessed.
    * Files excluded using `.helmignore` cannot be accessed.
* Charts do not preserve UNIX mode information, so file-level permissions will have no impact on the availability of a file when it comes to the `.Files` object.

### BASIC EXAMPLE

```bash
cat > config1.toml << EOF
message = Hello from config 1
EOF

cat > config2.toml << EOF
message = This is config 2
EOF

cat > config3.toml << EOF
message = Goodbye from config 3 
EOF

cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- \$files := .Files }}
  {{- range tuple "config1.toml" "config2.toml" "config3.toml" }}
  {{ . }}: |-
    {{ \$files.Get . }}
  {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: yummy-waterbuffalo-configmap
data:
  config1.toml: |-
    message = Hello from config 1

  config2.toml: |-
    message = This is config 2

  config3.toml: |-
    message = Goodbye from config 3
##
```

### GLOB PATTERNS

```bash
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- \$files := .Files }}
  {{- range \$path, \$bytes := \$files.Glob "**.toml" }}
  {{ \$path }}: |-
    {{ \$files.Get \$path }}
  {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: idle-heron-configmap
data:

  config1.toml: |-
    message = Hello from config 1

  config2.toml: |-
    message = This is config 2

  config3.toml: |-
    message = Goodbye from config 3
##
```

### ENCODING

```bash
cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- \$files := .Files }}
  {{- range \$path, \$bytes := \$files.Glob "**.toml" }}
  {{ \$path }}: |-
    {{ \$files.Get \$path | b64enc }}
  {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pruning-iguana-configmap
data:
  config1.toml: |-
    bWVzc2FnZSA9IEhlbGxvIGZyb20gY29uZmlnIDEK
  config2.toml: |-
    bWVzc2FnZSA9IFRoaXMgaXMgY29uZmlnIDIK
  config3.toml: |-
    bWVzc2FnZSA9IEdvb2RieWUgZnJvbSBjb25maWcgMyAK
##
```

### LINES

```bash
cat > toppings.txt << EOF
mushrooms
cheese
peppers
onions
EOF

cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $files := .Files }}
  {{- range $path, $bytes := $files.Glob "**.toml" }}
  {{ $path }}: |-
    {{ $files.Get $path | b64enc }}
  {{- end }}
  toppings: |-
    {{- range .Files.Lines "toppings.txt" }}
    {{ . }}
    {{- end }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: old-llama-configmap
data:
  config1.toml: |-
    bWVzc2FnZSA9IEhlbGxvIGZyb20gY29uZmlnIDEK
  config2.toml: |-
    bWVzc2FnZSA9IFRoaXMgaXMgY29uZmlnIDIK
  config3.toml: |-
    bWVzc2FnZSA9IEdvb2RieWUgZnJvbSBjb25maWcgMyAK
  toppings: |-
    mushrooms
    cheese
    peppers
    onions
##
```

## Creating a NOTES.txt File

```bash
cat > templates/NOTES.txt << EOF
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get {{ .Release.Name }}
EOF

helm install .
# Output
NAME:   excited-snail
LAST DEPLOYED: Sun Jan  6 19:33:19 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                     DATA  AGE
excited-snail-configmap  4     0s


NOTES:
Thank you for installing mychart.

Your release is named excited-snail.

To learn more about the release, try:

  $ helm status excited-snail
  $ helm get excited-snail
##
```

## Subcharts and Global Values

About subcharts.
1.  A subchart is considered “stand-alone”, which means a subchart can never explicitly depend on its parent chart.
2.  For that reason, a subchart cannot access the values of its parent.
3.  A parent chart can override values for subcharts.
4.  Helm has a concept of _global values_ that can be accessed by all charts.

```bash
# Creating subchart
cd mychart/charts

helm create mysubchart
# Output
Creating mysubchart
##

rm -rf mysubchart/templates/*.*
rm -rf mysubchart/templates/tests
```

### ADDING VALUES AND A TEMPLATE TO THE SUBCHART


```bash
cat > charts/mysubchart/values.yaml << EOF
dessert: cake
EOF

cat > charts/mysubchart/templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-cfgmap2
data:
  dessert: {{ .Values.dessert }}
EOF

helm install --dry-run --debug charts/mysubchart
# Output
[debug] Created tunnel using local port: '37419'

[debug] SERVER: "127.0.0.1:37419"

[debug] Original chart version: ""
[debug] CHART PATH: /home/anderson/dev/helm-tutorial/mychart/charts/mysubchart

NAME:   lazy-crocodile
REVISION: 1
RELEASED: Sun Jan  6 19:50:34 2019
CHART: mysubchart-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
dessert: cake

HOOKS:
MANIFEST:
##
```

## OVERRIDING VALUES FROM A PARENT CHART

```bash
cat > values.yaml << EOF
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
mysubchart:
  dessert: ice cream
EOF

helm install --dry-run --debug .
# Output
[debug] Created tunnel using local port: '42701'

[debug] SERVER: "127.0.0.1:42701"

[debug] Original chart version: ""
[debug] CHART PATH: /home/anderson/dev/helm-tutorial/mychart

NAME:   truculent-rattlesnake
REVISION: 1
RELEASED: Sun Jan  6 19:57:32 2019
CHART: mychart-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
favorite:
  drink: coffee
  food: pizza
mysubchart:
  dessert: ice cream
  global: {}
pizzaToppings:
- mushrooms
- cheese
- peppers
- onions

HOOKS:
MANIFEST:

---
# Source: mychart/charts/mysubchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: truculent-rattlesnake-cfgmap2
data:
  dessert: ice cream
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: truculent-rattlesnake-configmap
data:
  config1.toml: |-
    bWVzc2FnZSA9IEhlbGxvIGZyb20gY29uZmlnIDEK
  config2.toml: |-
    bWVzc2FnZSA9IFRoaXMgaXMgY29uZmlnIDIK
  config3.toml: |-
    bWVzc2FnZSA9IEdvb2RieWUgZnJvbSBjb25maWcgMyAK
  toppings: |-
    mushrooms
    cheese
    peppers
    onions
##
```

## GLOBAL CHART VALUES

```bash
cat > values.yaml << EOF
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions

mysubchart:
  dessert: ice cream

global:
  salad: caesar
EOF

cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- \$files := .Files }}
  {{- range \$path, \$bytes := \$files.Glob "**.toml" }}
  {{ \$path }}: |-
    {{ \$files.Get \$path | b64enc }}
  {{- end }}
  salad: {{ .Values.global.salad }}
  toppings: |-
    {{- range .Files.Lines "toppings.txt" }}
    {{ . }}
    {{- end }}
EOF

cat > charts/mysubchart/templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-cfgmap2
data:
  dessert: {{ .Values.dessert }}
  salad: {{ .Values.global.salad }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/charts/mysubchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alternating-buffalo-cfgmap2
data:
  dessert: ice cream
  salad: caesar
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alternating-buffalo-configmap
data:
  config1.toml: |-
    bWVzc2FnZSA9IEhlbGxvIGZyb20gY29uZmlnIDEK
  config2.toml: |-
    bWVzc2FnZSA9IFRoaXMgaXMgY29uZmlnIDIK
  config3.toml: |-
    bWVzc2FnZSA9IEdvb2RieWUgZnJvbSBjb25maWcgMyAK
  salad: caesar
  toppings: |-
    mushrooms
    cheese
    peppers
    onions
##
```

### SHARING TEMPLATES WITH SUBCHARTS

Parent charts and subcharts can share templates. Any defined block in any chart is available to other charts.

```bash
cat > templates/_helpers.tpl << EOF
{{- define "mychart.app" -}}
app_name: {{ .Chart.Name }}
app_version: "{{ .Chart.Version }}+{{ .Release.Time.Seconds }}"
{{- end -}}
EOF

cat > templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $files := .Files }}
  {{- range $path, $bytes := $files.Glob "**.toml" }}
  {{ $path }}: |-
    {{ $files.Get $path | b64enc }}
  {{- end }}
  salad: {{ .Values.global.salad }}
  labels: 
    {{- include "mychart.app" . | nindent 4 }}
  toppings: |-
    {{- range .Files.Lines "toppings.txt" }}
    {{ . }}
    {{- end }}
EOF

cat > charts/mysubchart/templates/configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-cfgmap2
  labels: 
    {{- include "mychart.app" . | nindent 4 }}
data:
  dessert: {{ .Values.dessert }}
  salad: {{ .Values.global.salad }}
EOF

helm install --dry-run --debug .
# Partial output
---
# Source: mychart/charts/mysubchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: banking-sponge-cfgmap2
  labels:
    app_name: mysubchart
    app_version: "0.1.0+1546815217"
data:
  dessert: ice cream
  salad: caesar
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: banking-sponge-configmap
data:
  config1.toml: |-
    bWVzc2FnZSA9IEhlbGxvIGZyb20gY29uZmlnIDEK
  config2.toml: |-
    bWVzc2FnZSA9IFRoaXMgaXMgY29uZmlnIDIK
  config3.toml: |-
    bWVzc2FnZSA9IEdvb2RieWUgZnJvbSBjb25maWcgMyAK
  salad: caesar
  labels:
    app_name: mychart
    app_version: "0.1.0+1546815217"
  toppings: |-
    mushrooms
    cheese
    peppers
    onions
##
```

## Debugging Templates

Debug:
* `helm lint` is your go-to tool for verifying that your chart follows best practices
* `helm install --dry-run --debug`: We’ve seen this trick already. It’s a great way to have the server render your templates, then return the resulting manifest file.
* `helm get manifest`: This is a good way to see what templates are installed on the server.


## References
* [The Chart Template Developer’s Guide](https://docs.helm.sh/chart_template_guide/#../charts)
* [Getting Started with a Chart Template](https://docs.helm.sh/chart_template_guide/)