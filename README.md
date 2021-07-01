# IrusAnalytics

IrusAnalytics is a gem that provides a simple way to send analytics to the IRUS-UK repository agggregation service.  

More information about IRUS-UK can be found at [https://irus.jisc.ac.uk/r5/](https://irus.jisc.ac.uk/r5/).  In summary the IRUS-UK service is designed to provide article-level usage statistics for Institutional Repositories.  To sign up and use IRUS-UK, please see the above link. 

This gem was developed for use with a Hydra repository [http://projecthydra.org/](http://projecthydra.org/), but it can be used with any other Rails based web application. 

Note: The University of Michigan have developed a new version of this Gem for IRUS R5 [https://github.com/mlibrary/irus_analytics](https://github.com/mlibrary/irus_analytics),

# Build Status
![Build Status](https://api.travis-ci.org/uohull/irus_analytics.png?branch=master)

## Installation

Add this line to your application's Gemfile:

    gem 'irus_analytics'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install irus_analytics

## Usage

Once you have the gem, run the following generator:

    $ rails g irus_analytics

This will generate a configuration file that exists within config/initializers/irus_analytics.rb

**IrusAnalytics.configuration.source\_repository** is used to configure the name of the source respository url (i.e. what the url for your repository)
**IrusAnalytics.configuration.irus\_server\_address** is used to define the IRUS-UK server endpoint, this can be configured for the test version of the service.  

The Irus analytics code is designed to be called after a download event has happened in your rails application.  The following code needs adding to the Rails controller handles the content download.

A simple example...

    class YourDownloadController < ApplicationController
      # You need to include the IrusAnalytics behaviour module
      include IrusAnalytics::Controller::AnalyticsBehaviour 

      after_filter :send_analytics, only: [:show]

      def show
        @id = params[:id] 
        # Your code
      end

      protected 
    
      #  You need to define this method for IrusAnalytics to use as the identifier (typically a OAI valid identifier)
      def item_identifier 
        @id
      end
    end

Therefore in summary...

    include IrusAnalytics::Controller::AnalyticsBehaviour  
  
    after_filter :send_analytics, only: [:show]

    def item_identifier
    end

... needs adding to the relevant controller.  

To be compliant with the IRUS-UK client requirements/recommendations this Gem makes use of the Resque  [https://github.com/resque/resque](https://github.com/resque/resque).  Resque provides a simple way to create background jobs within your Ruby application, and is specifically used within this gem to push the analytics posts onto a queue.  This means the download functionality within your application is unaffected by the send analytics call, and it provides a means of queuing analytics if the IRUS-UK server is down. 

Note: Resque requires Redis to be installed  

By installing this gem, your application should have access to the Resque rake tasks.  These can be seen by running "rake -T",  the list should include:-

    rake resque:failures:sort 
    rake resque:work
    rake resque:workers

To start the resque background job for this gem use

    QUEUE=irus_analytics rake environment resque:work


## Contributing

1. Fork it ( https://github.com/uohull/irus_analytics/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
