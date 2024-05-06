FROM ruby:2.7

RUN apt-get update -qq && apt-get install -y build-essential

ENV APP_HOME /app
WORKDIR $APP_HOME
ADD . $APP_HOME/
RUN bundle update --bundler && bundle install

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]
