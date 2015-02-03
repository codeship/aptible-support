require 'rspec/core/rake_task'

def system!(cmd)
  fail "Command failed: #{cmd}" unless system(cmd)
end

desc 'Tag current HEAD and push to release branch'
task :deploy, [:bucket] do |_t, args|
  fail 'No bucket specified' unless args[:bucket]

  # Build site
  system!('bundle exec middleman build')

  # Upload to S3
  opts = '--acl public-read ' \
         '--cache-control max-age=300,s-maxage=300'

  # http://makandracards.com/makandra/15649
  Bundler.with_clean_env do
    system!("aws s3 sync #{opts} build s3://#{args[:bucket]}")
  end
end

task :redirect, [:bucket] do |_t, args|
  fail 'No bucket specified' unless args[:bucket]

  system!('bundle exec middleman build')

  Bundler.with_clean_env do
    system!("S3_BUCKET=#{args[:bucket]} foreman run bundle exec middleman s3_redirect")
  end
end

namespace :deploy do
  desc 'Build and deploy site to support.aptible-staging.com'
  task :staging do
    Rake::Task[:deploy].invoke('support.aptible-staging.com')
    Rake::Task[:redirect].invoke('support.aptible-staging.com')
  end

  desc 'Build and deploy site to support.aptible.com'
  task :production do
    Rake::Task[:deploy].invoke('support.aptible.com')
    Rake::Task[:redirect].invoke('support.aptible.com')
  end
end

RSpec::Core::RakeTask.new(:spec)
task default: :spec