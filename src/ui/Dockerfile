FROM ruby:2.7
RUN apt-get update -qq && apt-get install -y build-essential

# FROM ubuntu:16.04
# RUN apt-get update \
#     && apt-get install -y ruby-full ruby-dev build-essential \
#     && gem install bundler --no-ri --no-rdoc

ENV APP_HOME /app
RUN mkdir $APP_HOME

WORKDIR $APP_HOME
ADD Gemfile* $APP_HOME/
RUN bundle update --bundler
RUN gem install bundler:1.17.2
RUN bundle install
ADD . $APP_HOME

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]

