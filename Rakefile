require "rubygems"
require "tmpdir"

require "bundler/setup"
require "jekyll"

# Change your GitHub reponame
GITHUB_REPONAME = "git@github.com:hoang-tran/hoang-tran.github.io.git"
publish_directory = "../hoang-tran-blog-publish"

desc "Generate blog files"
task :generate do
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end

desc "Prepare the publish directory, only need to execute once"
task :prepare_publish_directory do
  Dir.mkdir(publish_directory) unless File.exists?(publish_directory)
  
  pwd = Dir.pwd
  Dir.chdir publish_directory

  system "git init"
  system "git remote add origin #{GITHUB_REPONAME}"

  Dir.chdir pwd
end

desc "Generate and publish blog to master"
task :publish => [:generate] do
  Dir.mkdir(publish_directory) unless File.exists?(publish_directory)
  
  FileUtils.rm_rf Dir.glob("#{publish_directory}/*")

  cp_r "_site/.", publish_directory

  pwd = Dir.pwd
  Dir.chdir publish_directory

  system "git add ."
  message = "Site updated at #{Time.now.utc}"
  system "git commit -m #{message.inspect}"
  system "git push origin master --force"

  Dir.chdir pwd
end

drafts_dir = '_drafts'
posts_dir  = '_posts'

# rake post['my new post']
desc 'create a new post with "rake post[\'post title\']"'
task :post, :title do |t, args|
  if args.title
    title = args.title
  else
    puts "Please try again. Remember to include the filename."
  end
  mkdir_p "#{posts_dir}"
  filename = "#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.downcase.gsub(/[^\w]+/, '-')}.md"
  puts "Creating new post: #{filename}"
  File.open(filename, "w") do |f|
    f << <<-EOS.gsub(/^    /, '')
    ---
    layout: post
    title: #{title}
    date: #{Time.new.strftime('%Y-%m-%d %H:%M')}
    categories:
    ---

    EOS
  end

# Uncomment the line below if you want the post to automatically open in your default text editor
#  system ("#{ENV['EDITOR']} #{filename}")
end

# usage: rake draft['my new draft']
desc 'create a new draft post with "rake draft[\'draft title\']"'
task :draft, :title do |t, args|
  if args.title
    title = args.title
  else
    puts "Please try again. Remember to include the filename."
  end
  mkdir_p "#{drafts_dir}"
  filename = "#{drafts_dir}/#{title.downcase.gsub(/[^\w]+/, '-')}.md"
  puts "Creating new draft: #{filename}"
  File.open(filename, "w") do |f|
    f << <<-EOS.gsub(/^    /, '')
    ---
    layout: post
    title: #{title}
    date: #{Time.new.strftime('%Y-%m-%d %H:%M')}
    categories:
    ---

    EOS
  end

# Uncomment the line below if you want the draft to automatically open in your default text editor
# system ("#{ENV['EDITOR']} #{filename}")
end

desc 'preview the site with drafts'
task :preview do
  puts "## Generating site"
  puts "## Stop with ^C ( <CTRL>+C )"
  system "jekyll serve --watch --drafts"
end

desc 'list tasks'
task :list do
  puts "Tasks: #{(Rake::Task.tasks - [Rake::Task[:list]]).join(', ')}"
  puts "(type rake -T for more detail)\n\n"
end
