
https://github.com/norwoodj/helm-docs/tree/master?tab=readme-ov-file

The helm-docs tool auto-generates documentation from helm charts into markdown files.



The markdown generation is entirely gotemplate driven. The tool parses metadata from charts and generates a number of sub-templates that can be referenced in a template file (by default README.md.gotmpl). If no template file is provided, the tool has a default internal template that will generate a reasonably formatted README.

# 1 Installation 


## 1.1 mac 
helm-docs can be installed using [homebrew](https://brew.sh/):

```shell
brew install norwoodj/tap/helm-docs
```

## 1.2 powershell 



or [scoop](https://scoop.sh):

安装 scoop 首先 

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

https://stackoverflow.com/questions/74763204/installing-scoop-fails-running-the-installer-as-administrator-is-disabled-by-d
如果 在admin 模式下 Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression 报错 
则执行  iex "& {$(irm get.scoop.sh)} -RunAsAdmin"
```

再
```shell
scoop install helm-docs
```


helm-docs 安装在了 
`~\scoop\apps\helm-docs\current `=> `~\scoop\apps\helm-docs\1.14.2`
`c:\Users\yzh\scoop\apps\helm-docs`\

## 1.3 通过二进制文件安装 

This will download and install the latest release of the tool.

To build from source in this repository:

cd cmd/helm-docs
go build

## 1.4 通过go安装 

Or install from source:
    go install github.com/norwoodj/helm-docs/cmd/helm-docs@latest



# 2 Pre-commit hook

https://pre-commit.com/#install

If you want to automatically generate README.md files with a pre-commit hook, make sure you install the pre-commit binary, and add a .pre-commit-config.yaml file to your project. Then run:

Future changes to your chart's requirements.yaml, values.yaml, Chart.yaml, or README.md.gotmpl files will cause an update to documentation when you commit.

## 2.1 安装 

pre-commit install
pre-commit install-hooks


## 2.2 .pre-commit-config.yaml file 的写法

我用的是 

```
repos:
  - repo: local
    hooks:
      - id: helm-docs-container
        args: []
        description: Uses the container image of 'helm-docs' to create documentation from the Helm chart's 'values.yaml' file, and inserts the result into a corresponding 'README.md' file.
        entry: jnorwood/helm-docs:latest
        files: (README\.md\.gotmpl|(Chart|requirements|values)\.yaml)$
        language: docker_image
        name: Helm Docs Container
        require_serial: true

#repos:
#  - repo: https://github.com/norwoodj/helm-docs
#    rev: v1.14.2
#    hooks:
#      - id: helm-docs-container
#        entry: jnorwood/helm-docs:latest
#        args:
#          # Make the tool search for charts only under the current directory
#          - --chart-search-root=.
#          # Repeating the flag adds this to the list, now [./_templates.gotmpl, README.md.gotmpl]
#          # A base filename makes it relative to each chart directory found
#          - --template-files=README.md.gotmpl
```


Example 

```
repos:
  - repo: https://github.com/norwoodj/helm-docs
    rev: v1.14.2
    hooks:
      - id: helm-docs
        args:
          # Make the tool search for charts only under the `example-charts` directory
          - --chart-search-root=.

          # The `./` makes it relative to the chart-search-root set above
          - --template-files=./_templates.gotmpl

          # Repeating the flag adds this to the list, now [./_templates.gotmpl, README.md.gotmpl]
          # A base filename makes it relative to each chart directory found
          - --template-files=README.md.gotmpl
```

```yaml
repos:
  - repo: https://github.com/norwoodj/helm-docs
    rev: v1.14.2
    hooks:
      - id: helm-docs-container
        entry: jnorwood/helm-docs:latest
        args:
          # Make the tool search for charts only under the `example-charts` directory
          - --chart-search-root=example-charts

          # The `./` makes it relative to the chart-search-root set above
          - --template-files=./_templates.gotmpl

          # Repeating the flag adds this to the list, now [./_templates.gotmpl, README.md.gotmpl]
          # A base filename makes it relative to each chart directory found
          - --template-files=README.md.gotmpl
```



### 2.2.1 指定 helm-docs 的源位置

helm-docs Uses helm-docs binary located in your PATH
```
repos:
  - repo: https://github.com/norwoodj/helm-docs
    rev:  ""
    hooks:
      - id: helm-docs
        args:
          # Make the tool search for charts only under the `charts` directory
          - --chart-search-root=charts
```

helm-docs-built Uses helm-docs built from code in git
```
repos:
  - repo: https://github.com/norwoodj/helm-docs
    rev:  ""
    hooks:
      - id: helm-docs-built
        args:
          # Make the tool search for charts only under the `charts` directory
          - --chart-search-root=charts
```

helm-docs-container Uses the container image of helm-docs:latest
```
repos:
  - repo: https://github.com/norwoodj/helm-docs
    rev:  ""
    hooks:
      - id: helm-docs-container
        args:
          # Make the tool search for charts only under the `charts` directory
          - --chart-search-root=charts
```

To pin the helm-docs container to a specific tag, follow the example below:
```

---
repos:
  - repo: https://github.com/norwoodj/helm-docs
    rev:  ""
    hooks:
      - id: helm-docs-container
        entry: jnorwood/helm-docs:x.y.z
        args:
          # Make the tool search for charts only under the `charts` directory
          - --chart-search-root=charts

```


### 2.2.2 `--chart-search-root` and `--template-files`

There are two important parameters to be aware of when running helm-docs. 
--chart-search-root specifies the directory under which the tool will recursively search for charts to render documentation for. 
--template-files specifies the list of gotemplate files that should be used in rendering the resulting markdown file for each chart found. 
By default --chart-search-root=. and --template-files=README.md.gotmpl.


template file  的位置 通过相对路径给出 
If a template file is specified as a filename only as with the default above, the file is interpreted as being relative to each chart directory found. If however a template file is specified as a relative path,` e.g. the first of --template-files=./_templates.gotmpl --template-files=README.md.gotmpl` then the file is interpreted as being relative to the chart-search-root.

如果 template files 没有被给出， 则使用 internal default template
If any of the specified template files is not found for a chart (you'll notice most of the example charts do not have a README.md.gotmpl) file, then the internal default template is used instead.


## 2.3 .pre-commit-config.yaml 内容中使用其他写法

 清空 .pre-commit-config.yaml， 选 .pre-commit-hooks.yaml example 中的某一项， 添加到  .pre-commit-config.yaml  也有同样的效果 

https://github.com/norwoodj/helm-docs/blob/master/.pre-commit-hooks.yaml

就是不用写 `--chart-search-root` and `--template-files 了 

```
repos:
  - repo: local
    hooks:
      - id: helm-docs-container
        args: []
        description: Uses the container image of 'helm-docs' to create documentation from the Helm chart's 'values.yaml' file, and inserts the result into a corresponding 'README.md' file.
        entry: jnorwood/helm-docs:latest
        files: (README\.md\.gotmpl|(Chart|requirements|values)\.yaml)$
        language: docker_image
        name: Helm Docs Container
        require_serial: true
```


# 3 README.md.gotmpl

再 https://github.com/norwoodj/helm-docs/tree/master/example-charts/files-values 和 其他文件夹下可以找到 README.md.gotmpl 的例子 


我自己使用的 是 ivuplan-chart 中的 

```
{{ define "chart.valueDefaultColumnRender" }}
{{- $defaultValue := (default .Default .AutoDefault)  -}}
{{- $notationType := .NotationType }}
{{- if (and (hasPrefix "`" $defaultValue) (hasSuffix "`" $defaultValue) ) -}}
{{- $defaultValue = (toPrettyJson (fromJson (trimAll "`" (default .Default .AutoDefault) ) ) ) -}}
{{- $notationType = "json" }}
{{- end -}}
{{- if (eq $notationType "tpl" ) }}
<pre lang="{{ $notationType }}">
{{ .Key }}: |
{{- $defaultValue | nindent 2 }}
</pre>
{{- else if (eq $notationType "email") }}
<a href="mailto:{{ $defaultValue }}" style="color: green;">"{{ $defaultValue }}"</a>
{{- else }}
<pre lang="{{ $notationType }}">
{{ $defaultValue }}
</pre>
{{- end }}
{{ end }}

{{ define "chart.typeColumnRender" }}
{{- if (eq .Type "string/email") }}
<a href="#stringemail" title="{{- template "chart.valuetypes.email" -}}">{{.Type}}</a>
{{- else if (eq .Type "k8s/storage/persistent-volume/access-modes" )}}
<a target="_blank" 
   href="https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes"
   >{{- .Type }}</a>
{{- else }}
{{ .Type }}
{{- end }}
{{ end }}

{{ define "chart.valuesTableHtml" }}
<table>
	<thead>
		<th>Key</th>
		<th>Type</th>
		<th>Default</th>
		<th>Description</th>
	</thead>
	<tbody>
	{{- range .Values }}
		<tr>
			<td id="{{ .Key | replace "." "--" }}"><a href="./values.yaml#L{{ .LineNumber }}">{{ .Key }}</a></td>
			<td>{{- template "chart.typeColumnRender" . -}}</td>
			<td>
				<div style="max-width: 300px;">{{ template "chart.valueDefaultColumnRender" . }}</div>
			</td>
			<td>{{ if .Description }}{{ .Description }}{{ else }}{{ .AutoDescription }}{{ end }}</td>
		</tr>
	{{- end }}
	</tbody>
</table>
{{ end }}


# IVU.plan Helm Chart

![Version: {{ .Version }}](https://img.shields.io/badge/Version-{{ .Version | replace "-" "--" }}-informational?style=for-the-badge)
{{ if .Type }}![Type: {{ .Type }}](https://img.shields.io/badge/Type-{{ .Type }}-informational?style=for-the-badge) {{ end }}
{{ if .AppVersion }}![AppVersion: {{ .AppVersion }}](https://img.shields.io/badge/AppVersion-{{ .AppVersion | replace "-" "--" }}-informational?style=for-the-badge) {{ end }}

## Description

{{ template "chart.description" . }}

## Prerequisites

* Stakater Reloader (TODO: Add link)

## Usage

{{ template "chart.valuesSectionHtml" . }}

{{ template "chart.homepageLine" . }}

{{ template "chart.sourcesSection" . }}

{{ template "chart.maintainersSection" . }}

## Development

This document is generated with [helm-docs](https://github.com/norwoodj/helm-docs).
A pre-commit hook may be configured with [pre-commit](https://pre-commit.com/):


\```     # 使用的时候\ 需要被删除 
pre-commit install
\```      # 使用的时候\字符 需要被删除 

The pre-commit hook will run the helm-docs Docker container and ensure that the Readme is updated before issuing a commit.

```

# 4 使用 helm-docs 产生新的readme

1
Running the binary directly

To run and generate documentation into READMEs for all helm charts within or recursively contained by a directory:
helm-docs
helm-docs --dry-run # prints generated documentation to stdout rather than modifying READMEs

The tool searches recursively through subdirectories of the current directory for Chart.yaml files and generates documentation for every chart that it finds.


2
Using docker
You can mount a directory with charts under /helm-docs within the container.
Then run:
    docker run --rm --volume "$(pwd):/helm-docs" -u $(id -u) jnorwood/helm-docs:latest



# 5 Ignoring Chart Directories  (.helmdocsignore file )


helm-docs supports a .helmdocsignore file, exactly like a .gitignore file in which one can specify directories to ignore when searching for charts. Directories specified need not be charts themselves, so parent directories containing potentially many charts can be ignored and none of the charts underneath them will be processed. You may also directly reference the Chart.yaml file for a chart to skip processing for it.

