FROM couchdb

ENV ruby_ver="2.5.1"
ENV node_ver="8.11.3"

RUN apt-get update
RUN apt-get -y install git wget bzip2 autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev libxslt-dev libxml2-dev

# rbenvのインストール
RUN git clone https://github.com/sstephenson/rbenv.git /usr/local/rbenv
RUN git clone https://github.com/sstephenson/ruby-build.git /usr/local/rbenv/plugins/ruby-build
RUN echo 'export RBENV_ROOT="/usr/local/rbenv"' >> $HOME/.bashrc
RUN echo 'export PATH="${RBENV_ROOT}/bin:${PATH}"' >> $HOME/.bashrc
RUN echo 'eval "$(rbenv init -)"' >> $HOME/.bashrc
RUN /bin/bash -c "source $HOME/.bashrc; rbenv install ${ruby_ver}; rbenv global ${ruby_ver}"
RUN /bin/bash -c "source $HOME/.bashrc; gem update --system; gem install --no-ri --no-rdoc libxml-ruby couchrest"

# nvmのインストール
RUN curl --silent -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
RUN /bin/bash -c "source $HOME/.bashrc; nvm install $node_ver && nvm alias default $node_ver && nvm use default"

RUN mkdir -p /work
COPY scripts/* /work/

WORKDIR /work
