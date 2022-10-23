# Docker Best Practices

```
brew install colima
```

```
bin/prepare
```

```
bin/inspect Dockerfile.[variant]
```

```
bin/clean
```

### Variants
#### 👎 `Dockerfile.single-stage`

- `yarn install` and `bundle install` run in **serial**
- If any of the sources are changed, the installation is performed always.
```bash
=> [app  7/10] COPY . .
=> [app  8/10] RUN bundle install
=> [app  9/10] RUN yarn install
=> [app 10/10] RUN RAILS_ENV=production bundle exec rails assets:precompile  
 ```

#### 👎 `Dockerfile.separate-builder`

- `yarn install` and `bundle install` run in **serial**
- If `Gemfile.lock` is changed, `yarn install` is performed always.
```bash
=> [builder  6/15] COPY Gemfile Gemfile.lock ./
=> [builder  7/15] RUN bundle install
=> [builder  8/15] COPY package.json yarn.lock ./
=> [builder  9/15] RUN yarn install
```

#### 👍 `Dockerfile.separate-processes`

- `yarn install` and `bundle install` run in **parallel**
- If `package.json` is changed, `bundle install` will not be executed.

```bash
=> [yarn 3/4] COPY package.json yarn.lock ./
=> [yarn 4/4] RUN yarn install
=> CACHED [builder 1/2] RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash - && apt-get install -y nodejs
=> CACHED [builder 2/2] RUN npm install --global yarn@1.22.19
=> CACHED [assets 1/9] WORKDIR /tmp                          
=> CACHED [bundler 1/4] WORKDIR /tmp                         
=> CACHED [bundler 2/4] RUN gem install bundler              
=> CACHED [bundler 3/4] COPY Gemfile Gemfile.lock ./         
=> CACHED [bundler 4/4] RUN bundle install                                   
```
