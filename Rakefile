#############################################################################
#
# Modified jekyllrb Rakefile from commit 6cee2a87ccb4a3d09f27af6d5a672485687aa500.
# https://github.com/jekyll/jekyll/blob/master/Rakefile
#
#############################################################################

require 'rake'
require 'date'
require 'yaml'

$LOAD_PATH.unshift(File.join(File.dirname(__FILE__), *%w[lib]))
require 'jekyll/version'

#############################################################################
#
# Helper functions
#
#############################################################################

def normalize_bullets(markdown)
  markdown.gsub(/\n\s{2}\*{1}/, "\n-")
end

def linkify_prs(markdown)
  markdown.gsub(/#(\d+)/) do |word|
    "[#{word}]({{ site.repository }}/issues/#{word.delete("#")})"
  end
end

def linkify_users(markdown)
  markdown.gsub(/(@\w+)/) do |username|
    "[#{username}](https://github.com/#{username.delete("@")})"
  end
end

def linkify(markdown)
  linkify_users(linkify_prs(markdown))
end

def liquid_escape(markdown)
  markdown.gsub(/(`{[{%].+[}%]}`)/, "{% raw %}\\1{% endraw %}")
end

def custom_release_header_anchors(markdown)
  header_regexp = /^(\d{1,2})\.(\d{1,2})\.(\d{1,2}) \/ \d{4}-\d{2}-\d{2}/
  section_regexp = /^### \w+ \w+$/
  markdown.split(/^##\s/).map do |release_notes|
    _, major, minor, patch = *release_notes.match(header_regexp)
    release_notes
      .gsub(header_regexp, "\\0\n{: #v\\1-\\2-\\3}")
      .gsub(section_regexp) { |section| "#{section}\n{: ##{sluffigy(section)}-v#{major}-#{minor}-#{patch}}" }
  end.join("\n## ")
end

def sluffigy(header)
  header.gsub(/#/, '').strip.downcase.gsub(/\s+/, '-')
end

def remove_head_from_history(markdown)
  index = markdown =~ /^##\s+\d+\.\d+\.\d+/
  markdown[index..-1]
end

def converted_history(markdown)
  remove_head_from_history(
    custom_release_header_anchors(
      liquid_escape(
        linkify(
          normalize_bullets(markdown)))))
end

#############################################################################
#
# Post and page tasks
#
#############################################################################

namespace :post do
  desc "Create a new post"
  task :create do
    title = ENV["title"] || "new-post"
    begin
      slug = parameterize(title)
      puts slug
    rescue => e
      puts "Error: invalid characters in title"
      exit -1
    end

    begin
      date = ENV['date'] ? Date.parse(ENV['date']) : Date.today
    rescue => e
      puts "Error: date format must be YYYY-MM-DD"
      exit -1
    end

    filename = File.join("_posts", "#{date}-#{slug}.md")
    if File.exist?(filename)
      puts "Error: post already exists"
      exit -1
    end

    header = { "layout" => "post", "title" => title }
    content = header.to_yaml + "---\n"

    if IO.write(filename, content)
      puts "Post #{filename} created"
    else
      puts "Error: #{filename} could not be written"
    end
  end
end

namespace :page do
  desc "Create a new page"
  task :create do
    title = ENV["title"] || "new-page"
    begin
      slug = parameterize(title)
      puts slug
    rescue => e
      puts "Error: invalid characters in title"
      exit -1
    end

    folder = ENV["folder"] || "."

    filename = File.join(folder, "#{slug}.md")
    if File.exist?(filename)
      puts "Error: page already exists"
      exit -1
    end

    header = { "layout" => "page", "title" => title }
    content = header.to_yaml + "---\n"

    if IO.write(filename, content)
      puts "Page #{filename} created"
    else
      puts "Error: #{filename} could not be written"
    end
  end
end

#############################################################################
#
# Site tasks - http://jekyllrb.com
#
#############################################################################

namespace :site do
  desc "Generate and view the site locally"
  task :serve do
    puts "Running Jekyll..."
    sh "bundle exec jekyll serve --watch --baseurl ''"
  end

  desc "Generate site"
  task :build do
    puts "Building site..."
    sh "bundle exec jekyll build"
  end

  desc "Commit the local site to the stage-live branch on GitHub, which will publish to NFSN"
  task :stage do
    # Configure git if this is run in Travis CI
    if ENV["TRAVIS"]
      sh "git config --global user.name ${GIT_NAME}"
      sh "git config --global user.email ${GIT_EMAIL}"
    end
    
    # Switch branches and build the site.
    sh "git checkout -b stage-live"
    sh "bundle exec jekyll build --destination live/"

    sha = `git log`.match(/[a-z0-9]{40}/)[0]
    sh "git add live/"
    sh "git commit -m 'Bump to @#{sha}.'"
    sh "git push https://${GH_TOKEN}@github.com/woodrad/woodrad.github.io.git stage-live --quiet"
    puts "Pushed updated branch stage-live."
  end
end
