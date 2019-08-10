---
title: "Genserver Webcrawling in Pure Elixir"
date: 2019-03-13T23:40:01-04:00
---

A lot of people think that you need a full Phoenix application to create robust background processing applications. I guess coming from Rails where most people just reach for Rails by default, it's not that strange. Think about the last time you've written Ruby code _without_ using Rails.

Strange right?

Well in Elixir land, developers are trying to shake off the feeling that Phoenix is Elixir and Elixir is Phoenix. Phoenix is beautiful, but Elixir is also drop-dead gorgeous.

Let's create a nice web crawling application that periodically scrapes information.

{{< highlight bash >}}
cd ~/Work
mix new crawler
cd crawler/
{{< /highlight >}}

This is a barebones Elixir application. You'll notice a module automatically created for us:

{{< highlight elixir >}}
defmodule Crawler do
  @moduledoc """
  Documentation for Crawler.
  """

  @doc """
  Hello world.

  ## Examples

      iex> Crawler.hello()
      :world

  """
  def hello do
    :world
  end
end
{{< /highlight >}}

Let's create an application supervisor that will start whenever our application starts running. We can do this by using `Application`.

Create a `lib/application.ex` file:

{{< highlight elixir >}}
defmodule Crawler.Application do
  use Application

  def start(_type, _args) do
    children = [

    ]

    Supervisor.start_link(children, strategy: :one_for_one, name: Crawler.Supervisor)
  end
end
{{< /highlight >}}

So we have our foreman in charge of the work. This bad boy will make sure everything is up and running and nobody is caught slacking.

Let's make sure our supervisor actually starts by going into our `mix.exs` file and adding our module callback:

{{< highlight elixir >}}
# Run "mix help compile.app" to learn about applications.
def application do
  [
    extra_applications: [:logger],
    mod: {Crawler.Application, []}
  ]
end
{{< /highlight >}}

Now when we run our app, for example with `iex -S mix`, the supervisor will be up and running.

Notice though that our supervisor doesn't really have anything to supervise. He's a foreman with no workers. Let's change that!

We'll create a simple crawler that requests the HTML body of a URL:

{{< highlight elixir >}}
defmodule Crawlers.SomeWebsite do
  require Logger
  alias Helpers

  defp crawl do
    Logger.debug("[Crawlers.SomeWebsite] crawl/0")
    url = "https://sergio.dev"
    %{body: body} = Helpers.http_get(url)
    body
  end
end
{{< /highlight >}}

Then we'll create a GenServer that periodically calls our crawler:

{{< highlight elixir >}}
defmodule Schedulers.SomeWebsite do
  use GenServer
  require Logger

  def start_link(args) do
    GenServer.start_link(__MODULE__, args, name: __MODULE__)
  end

  def init(state) do
    schedule()
    {:ok, state}
  end

  def handle_info(:crawl, state) do
    Logger.debug("[Schedulers.SomeWebsite] :crawl")

    Crawlers.SomeWebsite.crawl()

    schedule()
    {:noreply, state}
  end

  defp schedule do
    Process.send_after(self(), :crawl, 15_000) # Every 15 seconds.
  end
end
{{< /highlight >}}

Awesome, our worker is ready to go! Let's let our Supervisor know about the new member of the team and ask him to supervise the process.

In `lib/application.ex` just add the genserver to the list of chidren.

{{< highlight elixir >}}
defmodule Crawler.Application do
  use Application

  def start(_type, _args) do
    children = [
      Schedulers.SomeWebsite
    ]

    Supervisor.start_link(children, strategy: :one_for_one, name: Crawler.Supervisor)
  end
end
{{< /highlight >}}

We're done!

Start your application with `iex -S mix` and you should see content coming in every 15 seconds.

In your terminal you can run the observer command to view the processes of your application. You'll see your application, then your supervisor and finally your supervisors children.

{{< highlight bash >}}
:observer.start
{{< /highlight >}}