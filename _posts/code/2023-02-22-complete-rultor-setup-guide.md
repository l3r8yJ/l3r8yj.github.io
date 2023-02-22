---
title: Сomplete (almost) guide to setup Rultor.
layout: post
date: '2023-02-22'
last_modified_at: '2023-02-22'
categories:
- ci/cd
- rultor
tags: blog-post
---
## Let's see...
Well, if you are reading this, you probably know what [Rultor](https://doc.rultor.com/) is. But for some reason you don't understand how to set it up. I also have a [template repository](https://github.com/l3r8yJ/elegant) in case I start a new project, it's already a bit set up. You can fork it and use.

Well, today i'm going to review two cases:
  - [Ruby](#ruby) project, `@merge` and `@release`
  - [Rust](#rust) project, `@merge` and `@release`

<br/>

<div id="ruby">
  <h2>Let's begin with configuration for Ruby project.</h2>
</div>

1) You need to go into [Rubygems settings](https://rubygems.org/settings/edit) of your account and set the `MFA` level to `UI and gem sign`.


2) Then you need to create an `API key` with `push` permission. Do this [here](https://rubygems.org/profile/api_keys).


3) You have to go and create a `private` repository with `rubygems.yml` file:
```yaml
:rubygems_api_key: # your key
:backtrace: true | false
:verbose: true | false
```
Also in this repo we create `.rultor.yml`:
```yaml
friends:
  - your_nickname/repo_name
```
This configuration allows `rultor` to share these secrets with the repositories you have listed in this file.


4) Now you have to go to your main repository and create a `.rultor.yml`:
```yaml
architect:
  - your_nickname
assets:
  # Here is important thing, in my repo config placed in `assets/` folder.
  # If you just put your config directly, path is gonna be like `l3r8yJ/repo#rubygems.yml`
  rubygems.yml: your_nickname/your_repo#path_to_config/rubygems.yml
install: |
  pdd -f /dev/null
  sudo bundle install --no-color "--gemfile=$(pwd)/Gemfile"
release:
  # If you want clarification, 
  # you can ask a question in the comments, I'll answer you.
  script: |-
    bundle exec rake
    rm -rf *.gem
    sed -i "s/0\.0\.0/${tag}/g" lib/version.rb
    git add lib/version.rb
    git commit -m "version set to ${tag}"
    gem build jini.gemspec
    chmod 0600 ../rubygems.yml
    gem push *.gem --config-file ../rubygems.yml
merge:
  script: |-
    bundle exec rake
```
All the work you describe in the `Rakefile` will be done before `merge` or `release`.
Also `lib/gemname/version.rb` should look like this:
```ruby
module YourModule
  # The version should only be 0.0.0
  VERSION = '0.0.0'.freeze
end
```

5) We are almost done. Now we need to go into the `.gemspec` file and do a little thing:
```ruby
Gem::Specification.new do |s|
  ...
  # Turn off the OTP code for pushing the gem
  s.metadata = { 'rubygems_mfa_required' => 'false' }
  ...
end
```

6) Looks like we're ready to go. Now create a test issue and run all the commands you want to use.

<div id="rust">
  <h2>Here it goes for Rust project.</h2>
</div>

The first three steps will be very similar.

1) Go to [creates.io](https://crates.io/settings/tokens) and get the API key.

2) Do the same thing as described in step `3` in Ruby, for example, let's create `crates-credentials` without the extension:
```toml
[registry]
token = "plase_your_token"
```

3) Go to your main repository. Here we also create a `.rultor.yml`:
```yaml
architect:
  - your_nickname
docker:
  # you can create your own container.
  image: yegor256/rultor-image:1.13.0
assets:
  # Here is important thing, in my repo config placed in `assets/` folder.
  # If you just put your config directly, path is gonna be like `l3r8yJ/repo#crates-credentials
  credentials: your_nickname/your_repo#path_to_config/crates-credentials
install: |
  pdd --file=/dev/null
merge:
  # describe your pipeline
  script: |
    cargo --color=never test -vv
    cargo --color=never fmt --check
    cargo doc --no-deps
    cargo clippy
release:
  script: |-
    [[ "${tag}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || exit -1
    sed -i -e "s/^version = \"0.0.0\"/version = \"${tag}\"/" Cargo.toml
    sed -i -e "s/0.0.0/${tag}/" src/lib.rs
    cargo --color=never test -vv -- --nocapture
    cargo --color=never fmt --check
    cargo clippy
    git commit -am "${tag}"
    ls -a ../
    mkdir -p ~/.cargo && cp ../credentials ~/.cargo
    cargo --color=never publish
```
4) Now open your `Cargo.toml` and set right `version`:
```toml
version = "0.0.0"
```

5) Nice, now we ready to go!

Thank you for your attention! If you find a mistake or something like this, if you just have a question, then write in the comments!


