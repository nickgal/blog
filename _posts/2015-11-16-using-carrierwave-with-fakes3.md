---
layout: post
title: "Using Carrierwave With FakeS3"
date: 2015-11-16T14:36:19-05:00
tags: [ruby, carrierwave, s3, fakes3]
---
Here's a quick walkthrough for using  [Carrierwave](https://github.com/carrierwaveuploader/carrierwave) with
[FakeS3](https://github.com/jubos/fake-s3).

We are going to use `fakes3.local` for the hostname, `3100` for the port and `my-bucket` for the s3 bucket name.

Install the `fakes3` gem:
{% highlight sh %}
gem install fakes3
{% endhighlight %}

Associate `fakes3.local` with your local computer:
{% highlight sh %}
# /etc/hosts
127.0.0.1 fakes3.local
{% endhighlight %}

Edit the carrierwave initializer:
{% highlight ruby %}
# config/initializers/carrierwave.rb
CarrierWave.configure do |config|
  config.fog_provider = 'fog/aws'
  config.fog_credentials = {
    provider:              'AWS',
    aws_access_key_id:     '',
    aws_secret_access_key: '',
    host:                  'fakes3.local',
    scheme:                'http',
    path_style:            true,
    port:                  3100
  }
  config.fog_directory  = 'my-bucket'
end
{% endhighlight %}

Finally, run fakes3:
{% highlight sh %}
fakes3 --root=tmp/fakes3_root --port=3100 --hostname=fakes3.local
{% endhighlight %}
