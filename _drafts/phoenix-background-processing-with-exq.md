---
layout: post
title: "Phoenix Background Processing With Exq"
---

Exq requires redis. If you are on a Mac and have homebrew you can `brew install redis`

## Getting Started

Creating our demo
{% highlight sh %}
mix phoenix.new phoenix_exq_demo
cd phoenix_exq_demo
mix ecto.create
{% endhighlight %}

I am using [Postgres.app](http://postgresapp.com/) which requires a minor change to the generated [Ecto](https://github.com/elixir-lang/ecto) config.

{% highlight elixir %}
# config/dev.exs
config :phoenix_exq_demo, PhoenixExqDemo.Repo,
  adapter: Ecto.Adapters.Postgres,
  # Remove username and password
  # Future Phoenix versions should have this addressed by
  # https://github.com/phoenixframework/phoenix/issues/1345
  # username: "postgres",
  # password: "postgres",
  database: "phoenix_exq_demo_dev",
  hostname: "localhost",
  pool_size: 10
{% endhighlight %}


## Adding Exq

Add exq as a mix dependency
{% highlight elixir %}
# mix.exs
defp deps do
  [{:phoenix, "~> 1.0.3"},
   {:phoenix_ecto, "~> 1.1"},
   {:postgrex, ">= 0.0.0"},
   {:phoenix_html, "~> 2.1"},
   {:phoenix_live_reload, "~> 1.0", only: :dev},
   {:cowboy, "~> 1.0"},
   {:exq, "~> 0.4.1"}]
end
{% endhighlight %}


Add Exq as an OTP application
{% highlight elixir %}
# mix.exs
def application do
  [mod: {PhoenixExqDemo, []},
   applications: [:phoenix, :phoenix_html, :cowboy, :logger,
                  :phoenix_ecto, :postgrex, :exq]]
end
{% endhighlight %}

Add Exq config (Note the host is a character list and not a string)
{% highlight elixir %}
# config/dev.exs
config :exq,
  host: '127.0.0.1',
  port: 6379,
  namespace: "exq"
{% endhighlight %}

## Create a worker

{% highlight elixir %}
# web/workers/demo_worker.ex
defmodule PhoenixExqDemo.DemoWorker do
  require Logger

  def perform do
    Logger.debug "Hello from Exq"
  end

end
{% endhighlight %}

## Putting it together

Fetch dependencies:
{% highlight sh %}
mix deps.get
{% endhighlight %}

Start redis
{% highlight sh %}
redis-server
{% endhighlight %}

Testing it
{% highlight sh %}
iex -S mix
{% endhighlight %}
{% highlight elixir %}
iex> {:ok, ack} = Exq.enqueue(:exq, "default", PhoenixExqDemo.DemoWorker, [])
{:ok, "cf63bd50-1f0f-4b8b-a47c-0bf90d566473"}
[debug] Hello from Exq
{% endhighlight %}

## Bonus

Mounting the Exq webui with Phoenix.

{% highlight elixir %}
# web/router.ex
forward "/", Exq.RouterPlug, namespace: "exq"
scope "/", PhoenixExqDemo do
  pipe_through :browser # Use the default browser stack

  get "/", PageController, :index
end
{% endhighlight %}
