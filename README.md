# JSON-e Terraform Provider

A provider with a single resource `jsone_template`. This works similarly to the `template_file` resource
but allows us to use json-e constructs like $map to transforms lists and such things.

You can use [the json-e documentation](https://taskcluster.github.io/json-e/) to find out more about
how to use the templates.

## Usage

First `go get -u github.com/taskcluster/terraform-provider-jsone`. At that point, if you have built terraform from
source, simply running `go install` for this package will work. Otherwise, follow along with
[the official docs](https://www.terraform.io/docs/configuration/providers.html#third-party-plugins) on how to use
third party plugins.

Now it is as simple as:

```terraform
locals {
  context = {
    foo = "bar"
    baz = ["bing", "qux"]
  }
}

data "jsone_template" "service" {
  template = "${file("${path.module}/service.yaml")}"
  yaml_context = "${jsonencode(local.context)}"
}

output "rendered" {
  value = "${data.jsone_template.service.rendered}"
}
```

The resource takes a string called `template` and either a field called `context` or `yaml_context`.
These are mutually exclusive. `context` should be good enough for most cases and is just a terraform map,
but if you run into issues with how terraform handles mixed-type maps or need to pass in typed data, opt
for `yaml_context` which just takes a string containing yaml. Oftentimes using the terraform interpolation
of `jsonencode` will be your best bet.

You can access the rendered template with `.rendered` much the same as with `template_file`. This will normally
be in `json` format unless you explicitely pass in `format = "yaml"` to the resource.
