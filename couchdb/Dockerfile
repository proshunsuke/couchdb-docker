FROM couchdb

ENV ruby_ver="2.5.1"

RUN apt-get update
RUN apt-get -y install git wget bzip2 autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev libxslt-dev libxml2-dev
RUN git clone https://github.com/sstephenson/rbenv.git /usr/local/rbenv
RUN git clone https://github.com/sstephenson/ruby-build.git /usr/local/rbenv/plugins/ruby-build

RUN echo 'export RBENV_ROOT="/usr/local/rbenv"' >> $HOME/.bashrc
RUN echo 'export PATH="${RBENV_ROOT}/bin:${PATH}"' >> $HOME/.bashrc
RUN echo 'eval "$(rbenv init -)"' >> $HOME/.bashrc
RUN source $HOME/.bashrc

RUN rbenv install ${ruby_ver}; rbenv global ${ruby_ver}
RUN gem update --system; gem install --no-ri --no-rdoc libxml-ruby couchrest; gem install bundler

RUN mkdir -p /work
COPY scripts/import_from_jamendo.rb /work

WORKDIR /work
