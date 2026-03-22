---
title: "Extension Dev Quickstart"
parent: Getting Started
nav_order: 2
description: "Scaffold a working LegionIO extension in 10 minutes. Runners, actors, specs, and standalone client."
---

# Extension Dev Quickstart

**Time:** 10 minutes
**You'll build:** A working extension gem with a runner, a polling actor, and passing specs.
**Prerequisites:** Ruby >= 3.4, Bundler

{: .note }
> This quickstart shows the extension system (LEX) — how LegionIO grows through composable gems. By the end, you'll have a working extension that integrates with the framework automatically.

## Step 1: Install

```bash
gem install legionio
```

## Step 2: Scaffold

```bash
legionio lex create weather_checker
```

```
  » Creating lex-weather_checker (template: basic)...
  » Files generated
  » Git initialized
  » Bundle installed

  » Extension lex-weather_checker created in ./lex-weather_checker

  Next steps:
    cd lex-weather_checker
    # Add runners:  legionio generate runner my_runner
    # Add actors:   legionio generate actor my_actor
```

```bash
cd lex-weather_checker
```

The scaffold creates a complete gem layout:

```
lex-weather_checker/
  lex-weather_checker.gemspec
  Gemfile
  .gitignore
  .rubocop.yml
  LICENSE
  README.md
  lib/
    legion/
      extensions/
        weather_checker/
          runners/
          actors/
          tools/
            .gitkeep
          version.rb
          client.rb
        weather_checker.rb
  spec/
    spec_helper.rb
    legion/
      extensions/
        weather_checker_spec.rb
  .github/
    workflows/
      rspec.yml
      rubocop.yml
```

The `template: basic` is the default. Other available templates are: `llm-agent`, `service-integration`, `data-pipeline`, `scheduled-task`, and `webhook-handler`. Pass `--template` to select one.

## Step 3: Add a Runner

Runners hold your business logic — callable functions that do the actual work.

```bash
legionio generate runner forecast
```

```
  » Created lib/legion/extensions/weather_checker/runners/forecast.rb
  » Created spec/runners/forecast_spec.rb

  Functions scaffolded: execute
  Add actors with: legionio generate actor forecast --type subscription
```

Edit the generated runner to add your logic:

```ruby
# lib/legion/extensions/weather_checker/runners/forecast.rb
module Legion
  module Extensions
    module WeatherChecker
      module Runners
        module Forecast
          def get(city:, **)
            connection = Faraday.new("https://wttr.in")
            response = connection.get("/#{city}?format=j1")
            Legion::JSON.load(response.body)
          end
        end
      end
    end
  end
end
```

## Step 4: Add a Polling Actor

Actors define how a runner is invoked. A polling actor calls your runner on a fixed interval.

```bash
legionio generate actor checker
```

```
  » Created lib/legion/extensions/weather_checker/actors/checker.rb
  » Created spec/actors/checker_spec.rb
```

Configure it to poll every 60 seconds:

```ruby
# lib/legion/extensions/weather_checker/actors/checker.rb
module Legion
  module Extensions
    module WeatherChecker
      module Actors
        class Checker < Legion::Extensions::Actors::Poll
          def action
            result = run_runner(Legion::Extensions::WeatherChecker::Runners::Forecast,
                               :get, city: "Minneapolis")
            Legion::Logging::Logger.info "Weather: #{result}"
          end

          def poll_interval
            60
          end
        end
      end
    end
  end
end
```

## Step 5: Run Specs

```bash
bundle exec rspec
```

```
Legion::Extensions::WeatherChecker
  is a valid extension
  has a version
  loads without error

Finished in 0.00412 seconds (files took 0.31 seconds to load)
3 examples, 0 failures
```

The scaffold ships with passing specs out of the box. Add your own for the runner logic in `spec/runners/forecast_spec.rb`.

## Step 6: Lint

```bash
bundle exec rubocop
```

```
Inspecting 6 files
......

6 files inspected, no offenses detected
```

## Step 7: Run It

Add the gem to a LegionIO project's Gemfile:

```ruby
gem "lex-weather_checker", path: "../lex-weather_checker"
```

Then start the engine:

```bash
legionio start
```

LegionIO auto-discovers extensions by scanning `Bundler.load.specs` at boot. No registration, no config — add the gem to the Gemfile and it's live. The `Checker` polling actor starts immediately and calls the `Forecast` runner every 60 seconds.

## What Just Happened?

You built a complete extension in 10 minutes:

- **Runner**: a callable function that fetches weather data from a live HTTP API
- **Actor**: a polling loop that calls the runner every 60 seconds
- **Transport**: AMQP wiring generated automatically by the framework
- **Specs**: test scaffolding included and passing from the first scaffold

The extension system is how LegionIO grows. The core stays small. Your extension composes with everything else — other extensions can call your runners, chain tasks through conditioners and transformers, or subscribe to results over AMQP.

## What's Next

- [Extension Catalog]({% link extensions.md %}) — see what's already built
- [Architecture Overview]({% link architecture.md %}) — understand runners, actors, and transport in depth
- [Cognitive Agent Quickstart]({% link getting-started/quickstart-agent.md %}) — see the cognitive architecture in action
- [Contributing Guide](https://github.com/LegionIO/.github/blob/main/CONTRIBUTING.md) — share your extension with the community
