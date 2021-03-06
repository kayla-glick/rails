*   Don't run `after_enqueue` and `after_perform` callbacks if the callback chain is halted.

        class MyJob < ApplicationJob
          before_enqueue { throw(:abort) }
          after_enqueue { # won't enter here anymore }
        end

    `after_enqueue` and `after_perform` callbacks will no longer run if the callback chain is halted.
    This behaviour is a breaking change and won't take effect until Rails 6.2.
    To enable this behaviour in your app right now, you can add in your app's configuration file
    `config.active_job.skip_after_callbacks_if_terminated = true`

    *Edouard Chin*

*   Fix enqueuing and performing incorrect logging message.

    Jobs will no longer always log "Enqueued MyJob" or "Performed MyJob" when they actually didn't get enqueued/performed.

    ```ruby
      class MyJob < ApplicationJob
        before_enqueue { throw(:abort) }
      end

      MyJob.perform_later # Will no longer log "Enqueued MyJob" since job wasn't even enqueued through adapter.
    ```

    A new message will be logged in case a job couldn't be enqueued, either because the callback chain was halted or
    because an exception happened during enqueing. (i.e. Redis is down when you try to enqueue your job)

    *Edouard Chin*

*   Add an option to disable logging of the job arguments when enqueuing and executing the job.

        class SensitiveJob < ApplicationJob
          self.log_arguments = false

          def perform(my_sensitive_argument)
          end
        end

    When dealing with sensitive arguments as password and tokens it is now possible to configure the job
    to not put the sensitive argument in the logs.

    *Rafael Mendonça França*

*   Changes in `queue_name_prefix` of a job no longer affects all other jobs.

    Fixes #37084.

    *Lucas Mansur*

*   Allow `Class` and `Module` instances to be serialized.

    *Kevin Deisz*

*   Log potential matches in `assert_enqueued_with` and `assert_performed_with`.

    *Gareth du Plooy*

*   Add `at` argument to the `perform_enqueued_jobs` test helper.

    *John Crepezzi*, *Eileen Uchitelle*

*   `assert_enqueued_with` and `assert_performed_with` can now test jobs with relative delay.

    *Vlado Cingel*

*   Add jitter to :exponentially_longer

    ActiveJob::Exceptions.retry_on with :exponentially_longer now uses a random amount of jitter in order to
    prevent the [thundering herd effect.](https://en.wikipedia.org/wiki/Thundering_herd_problem).  Defaults to
    15% (represented as 0.15) but overridable via the `:jitter` option when using `retry_on`.
    Jitter is applied when an `Integer`, `ActiveSupport::Duration` or `exponentially_longer`, is passed to the `wait` argument in `retry_on`.

    retry_on(MyError, wait: :exponentially_longer, jitter: 0.30)

    *Anthony Ross*


Please check [6-0-stable](https://github.com/rails/rails/blob/6-0-stable/activejob/CHANGELOG.md) for previous changes.
