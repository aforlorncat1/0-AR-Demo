require 'standalone_migrations'
StandaloneMigrations::Tasks.load_tasks

desc 'Start IRB with application environment loaded'
task "console" do
  exec "irb -r ./config/environment"
end