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

<!-- TODO: Fill in with tested end-to-end walkthrough -->
<!-- This is the #2 priority quickstart — Ruby developers are the home turf audience -->

## Step 1: Install

```bash
gem install legionio
```

## Step 2: Scaffold

```bash
legion lex create weather_checker
cd lex-weather_checker
```

Look at what was generated:

```
lex-weather_checker/
  lib/legion/extensions/weather_checker/
    runners/         # Business logic (callable functions)
    actors/          # Execution modes (subscription, polling, etc.)
    helpers/         # Shared utilities, client connections
    transport/       # AMQP exchanges, queues, messages
  spec/              # RSpec tests
  CHANGELOG.md
  README.md
  lex-weather_checker.gemspec
```

## Step 3: Add a Runner

```bash
legion generate runner forecast
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
            # Your business logic here
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

```bash
legion generate actor checker
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
bundle install
bundle exec rspec
```

The scaffold includes passing specs out of the box. Add your own for the runner logic.

## Step 6: Run It

Add the gem to a LegionIO project's Gemfile and start the engine:

```bash
legion start
```

Your extension is auto-discovered and the polling actor starts running. No registration, no config — drop the gem in and it's live.

## What Just Happened?

You built a complete extension in 10 minutes:
- **Runner**: a callable function that fetches weather data
- **Actor**: a polling loop that calls the runner every 60 seconds
- **Transport**: AMQP wiring generated automatically
- **Specs**: test scaffolding included

The extension system is how LegionIO grows. The core stays small. Your extension composes with everything else.

## What's Next

- [Extension Catalog]({% link extensions.md %}) — see what's already built
- [Architecture Overview]({% link architecture.md %}) — understand runners, actors, and transport in depth
- [Cognitive Agent Quickstart]({% link getting-started/quickstart-agent.md %}) — see the cognitive architecture in action
- [Contributing Guide](https://github.com/LegionIO/.github/blob/main/CONTRIBUTING.md) — share your extension with the community
