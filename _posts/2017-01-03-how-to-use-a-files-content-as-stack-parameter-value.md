---
layout: post
title: How to use a files content as stack parameter value
---

Sometimes you may want to configure your stack with data contained within a file or a file-like thing (S3 object f.e.).

## Possible usecases: 
* Your CI/CD pipeline builds a new artifact and supplies your artifacts name and version in a textfile.
* You have a somehow managed json file in an S3 bucket containing AWS account metadata for all accounts you own and want to get a list of account ids from it to grant permission to a shared resource.

## How it works

### What you can do in your cfn-sphere stack config file:

Cfn-Sphere reads plain content (in utf-8) from any file and takes it as parameter value. <br/>Furthermore it allows to make **JMESPath queries on YAML or JSON files** ([jmespath.org](http://jmespath.org)).

{% highlight YAML %}
region: eu-west-1
stacks:
  my-app:
    template-url: templates/my-app.yml
    parameters:
      dockerImageVersion: "|File|artifactVersion.txt"
  my-distribution-bucket:
    template-url: templates/bucket.yml
    parameters:
      trustedAccountIds: "|File|s3://myBucket/accounts.json|accounts.[].id"
{% endhighlight %}

#### Example content of artifactVersion.txt:
{% highlight ruby %}
myOrg/myApp:234
{% endhighlight %}

#### Example content of accounts.json:
{% highlight JSON %}
{
    "accounts": [{
        "name": "account-a",
        "id": "123325434123"
    }, {
        "name": "account-b",
        "id": "345123123123"
    }]
}
{% endhighlight %}

### The result as if you would not have used the macro:
{% highlight YAML %}
region: eu-west-1
stacks:
  my-app:
    template-url: templates/my-app.yml
    parameters:
      dockerImageVersion: myOrg/myApp:234
  my-distribution-bucket:
    template-url: templates/bucket.yml
    parameters:
      trustedAccountIds: ["123325434123", "345123123123"] 
{% endhighlight %}

