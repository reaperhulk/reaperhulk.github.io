---
author: admin
comments: true
layout: post
slug: git-hooks-for-pre-commit-rspec-testing
title: Git Hooks For Pre-Commit Rspec Testing
wordpress\_id: 1554
---

Here's a pair of scripts for automatically running Ruby rspec tests, refusing to commit if they fail, and pulling in the results into the commit msg. If you want to use them place them in your project's .git/hooks directory as executable scripts.

These scripts assume you have rspec writing to rspec\_results.html. To do this by default add a .rspec file in your project dir with "-f doc -f h -o rspec\_results.html" inside.

pre-commit

```ruby
#!/usr/bin/env ruby
require 'pty'
html_path = "rspec_results.html"
#PTY.spawn trick from
#http://stackoverflow.com/questions/1154846/continuously-read-from-stdout-of-external-process-in-ruby
begin
	PTY.spawn( "rake spec" ) do |stdin, stdout, pid|
	begin
		stdin.each { |line| print line }
	rescue Errno::EIO
		end
	end
rescue PTY::ChildExited
	puts "The child process exited!"
end

# find out how many errors were found
html = open(html_path).read
examples = html.match(/(\d+) examples/)[0].to_i rescue 0
failures = html.match(/(\d+) failures/)[0].to_i rescue 0
if failures == 0 then
	failures = html.match(/(\d+) failure/)[0].to_i rescue 0
end
pending = html.match(/(\d+) pending/)[0].to_i rescue 0

if failures.zero?
  puts "0 failed! #{examples} run, #{pending} pending"
  puts "View rspec results at #{File.expand_path(html_path)}"
  sleep 1
  exit 0
else
  puts "\aDID NOT COMMIT YOUR FILES!"
  puts "View rspec results at #{File.expand_path(html_path)}"
  puts
  puts "#{failures} failed! #{examples} run, #{pending} pending"
  `open #{html_path}`
  exit 1
end

```


prepare-commit-msg

```ruby
#!/usr/bin/env ruby
html_path = "rspec_results.html"
# find out how many errors were found
html = open(html_path).read
examples = html.match(/(\d+) examples/)[0].to_i rescue 0
failures = html.match(/(\d+) failures/)[0].to_i rescue 0
if failures == 0 then
        failures = html.match(/(\d+) failure/)[0].to_i rescue 0
end
pending = html.match(/(\d+) pending/)[0].to_i rescue 0


message_file = ARGV[0]
message = File.read(message_file)
index = message.index('# Please enter the commit message for your changes. Lines starting')
message.insert(index-1, "\n\n#{failures} failed! #{examples} run, #{pending} pending")
open(message_file,'w') { |f|
        f.puts(message)
}
```

