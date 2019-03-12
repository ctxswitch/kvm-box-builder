require 'yaml'
require 'fileutils'
require 'vagrant_cloud'

namespace :build do
  desc 'ubuntu'
  task :ubuntu do
    check_build_env
    packer_build 'ubuntu', 'ubuntu-16.04-amd64.json'
  end
end

namespace :release do
  desc 'release'
  task :ubuntu do
    check_release_env
    release_box 'ubuntu-16.04'
  end
end

desc 'clean'
task :clean do
  puts 'Removing bento/.kitchen.yml and bento/builds/*'
  FileUtils.rm_rf(['bento/.kitchen.yml', Dir.glob('bento/builds/*')])
end

desc 'clean_cache'
task :clean_cache do
  puts 'Removing downloaded isos in the packer cache'
  FileUtils.rm_rf([Dir.glob('bento/**/packer_cache/*')])
end

def check_build_env
  unless which 'packer'
    puts 'Please install packer before building boxes'
    exit 1
  end
end

def check_release_env
  if ENV['VAGRANT_CLOUD_TOKEN'].nil?
    puts 'Please set the VAGRANT_CLOUD_TOKEN environment variable'
    exit 1
  end

  if ENV['VAGRANT_CLOUD_ORG'].nil?
    puts 'Please set the VAGRANT_CLOUD_ORG environment variable'
    exit 1
  end
end

def packer_build(os, template)
  cmd = "packer build -only=qemu #{template}"

  FileUtils.cd "bento/#{os}" do
    sh cmd do |ok, res|
      unless ok
        puts "Packer build failed with status code: #{res.exitstatus}"
        exit 1
      end
    end
  end
end

def release_box(name)
  account = VagrantCloud::Account.new(ENV['VAGRANT_CLOUD_ORG'], ENV['VAGRANT_CLOUD_TOKEN'])
  box = account.ensure_box(name, short_description: "qemu bento box for #{name}", is_private: false)
  version = box.ensure_version('1.0.0', metadata(name, '1.0.0', '20190310114800').to_s)
  provider = version.ensure_provider('qemu', nil)
  provider.upload_file("bento/builds/#{name}.libvirt.box")
  # version.release
end

def which(executable)
  path = ENV['PATH'].split(':')
  path.any? { |p| File.exist?("#{p}/#{executable}") }
end

def metadata(name, version, ts)
  {
    name: name,
    version: version,
    build_timestamp: ts,
    git_revision: `git rev-parse HEAD`.strip,
    git_status: `git status --porcelain`.strip.empty? ? 'clean' : 'dirty',
    box_basename: "#{name}-#{version}",
    packer: `packer --version`.split('\n')[0],
    vagrant: `vagrant --version`.split(' ')[1],
  }
end
