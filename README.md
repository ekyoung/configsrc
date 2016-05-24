# configsrc

Go package for loading configuration information from an external source.

## Motivation

TODO

## Installation

```go
$ go get github.com/ekyoung/configsrc
```

## Usage

The general pattern is to:

1. Define a struct that represents all the configuration for your app.
1. Write a YAML file for each configuration you want to be able to load.
1. Set environment variables that tell configsrc where it should load a specific configuration file from.
1. Run your app.

`configsrc` looks for environment variables with a prefix containing a value you specify, plus `_CONFIGSRC_`.
For example, given the prefix "my_app", `configsrc` looks for environment variables starting with
"MY_APP_CONFIGSRC_". The remainder of the environment variable names vary for each supported type
of source and are described below.

**Sample main.go**

```go
package main

import (
	"log"

	"github.com/ekyoung/configsrc"
)

type AppConfig struct {
	MongoConnection string `yaml:"mongo_connection"`
	Mode            string
	Frequency       int
}

func main() {
	cfg := AppConfig{}

	err := configsrc.Load("my_app", &cfg)
	if err != nil {
		log.Fatalf("error: %v\n", err)
		return
	}

	log.Printf("[DEBUG] Config: %v\n", cfg)
}

```

YAML files are loaded with [go-yaml](https://github.com/go-yaml/yaml). See the readme in that repo
for struct tag options.

**Sample config file**

```yaml
mongo_connection: mongodb://localhost/my-db
mode: Active
frequency: 5
```

### S3

`configsrc` environment variables:

* `S3_BUCKET`
* `S3_KEY`

Additionaly, you should set the environment variables expected by the [AWS SDK](https://github.com/aws/aws-sdk-go)
in order to access your S3 resources.

**Example**

Given a prefix of "my_app", and an AWS Profile named "ACME-Admin" stored in the
[central credentials file](https://blogs.aws.amazon.com/security/post/Tx3D6U6WSFGOK2H/A-New-and-Standardized-Way-to-Manage-Credentials-in-the-AWS-SDKs),
set the following environment variables:

```
MY_APP_CONFIGSRC_S3_BUCKET=any-bucket-name
MY_APP_CONFIGSRC_S3_KEY=path/to/config.yaml
AWS_PROFILE=ACME-Admin
AWS_REGION=us-east-1
```