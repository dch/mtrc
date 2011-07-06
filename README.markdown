Mtrc
====
(Metric, for short)
----

A small library to accumulate metrics and extract basic statistics from them.
Want a latency profile of your Rack app? Boom:

    gem install mtrc

    class MyMetrics
      def initialize(app)
        @app = app
        @m = Mtrc::SortedSamples.new

        Thread.new do
          sleep 100
          puts <<EOF
    Min:      #{@m.min}
    Median:   #{@m.median}
    95th %:   #{@m % 95}
    99th %:   #{@m % 99}
    99.9th %: #{@m.at .999}
    Max:      #{@m.max}
    EOF
          @m = Mtrc::SortedSamples.new
        end
      end

      def call(env)
        t1 = Time.now
        r = @app.call env
        dt = Time.now - t1

        @m << dt
        r
      end        
    end

Which requests take the longest?

    @m << Mtrc::Sample.new dt, env[:PATH_INFO]
    (@m % 95).value # => "the 95th percentile request path"