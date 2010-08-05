require "rubygems"
require "sinatra"
require "spec/rake/spectask"
begin
  require "vlad"
  Vlad.load(:scm => :git, :app => nil, :web => nil)
rescue LoadError
end

require File.join(File.dirname(__FILE__), *%w[spec model_factory])
require File.join(File.dirname(__FILE__), *%w[lib models])
require File.join(File.dirname(__FILE__), *%w[lib config])

desc "Run all specs in spec directory"
Spec::Rake::SpecTask.new(:spec) do |t|
  t.spec_files = FileList["spec/*_spec.rb"]
end

class Factory
  include ModelFactory
end

class String
  def slugize
    self.downcase.gsub(/[^a-z0-9\-]/, '-').squeeze('-').gsub(/^\-/, '').gsub(/\-$/, '')
  end
end

def ask(q)
  print "#{q} "
  STDIN.gets.strip.chomp
end

namespace :heroku do
  desc "Set Heroku config vars from config.yml"
  task :config do
    Sinatra::Application.environment = ENV["RACK_ENV"] || "production"
    settings = {}
    Nesta::Config.settings.map do |variable|
      value = Nesta::Config.send(variable)
      settings["NESTA_#{variable.upcase}"] = value unless value.nil?
    end
    if Nesta::Config.author
      %w[name uri email].map do |author_var|
        value = Nesta::Config.author[author_var]
        settings["NESTA_AUTHOR__#{author_var.upcase}"] = value unless value.nil?
      end
    end
    params = settings.map { |k, v| %Q{#{k}="#{v}"} }.join(" ")
    system("heroku config:add #{params}")
  end
end

namespace :setup do
  desc "Create the content directory"
  task :content_dir do
    FileUtils.mkdir_p(Nesta::Config.content_path)
  end
  
  desc "Create some sample pages"
  task :sample_content => :content_dir do
    factory = Factory.new
    
    File.open(Nesta::Config.content_path("menu.txt"), "w") do |file|
      file.puts("fruit")
    end
    
    factory.create_category(
      :heading => "Fruit",
      :path => "fruit",
      :content => <<-EOF
This is a category page about Fruit. You can find it here: `#{File.join(Nesta::Config.page_path, 'fruit.mdown')}`.

The general idea of category pages is that you assign articles to categories (in a similar way that many CMS or blog systems allow you to tag your articles), and then write some text introductory text on the topic. So this paragraph really ought to have some introductory material about fruit in it. Why bother? Well it's a great way to structure your site and introduce a collection of articles. It can also help you to [get more traffic](http://www.wordtracker.com/academy/website-structure).

The articles assigned to the Fruit category are listed below, typically with a short summary of the article and a link inviting you to "read more". If you'd rather show the entire text of your article inline you can just omit the summary from your article.

Have a look at the files that define the articles that follow, and you should find that you can get the hang of it very quickly.
      EOF
    )

    %w[Pears Apples Bananas].each do |fruit|
      summary = <<-EOF
This would be an article on #{fruit} if I'd bothered to research it.
EOF
      file = File.join(Nesta::Config.page_path, "#{fruit.downcase}.mdown")
      location = "You can change this article by editing `#{file}`."
      factory.create_article(
        :heading => "The benefits of #{fruit}",
        :path => fruit.downcase,
        :metadata => {
          "date" => (Time.new - 10).to_s,
          "categories" => "fruit",
          "read more" => "Read more about #{fruit}",
          "summary" => summary.chomp
        },
        :content => "#{fruit} are good because... blah blah.\n\n#{location}"
      )
    end
    
    summary = <<-EOF
You can edit this article by opening `#{File.join(Nesta::Config.page_path, 'example.mdown')}` in your text editor. Make some changes, then save the file and reload this web page to preview your changes.
    EOF

    factory.create_article(
      :heading => "Getting started",
      :path => "getting-started",
      :metadata => {
        "date" => Time.new.to_s,
        "read more" => "Read more tips on getting started",
        "summary" => summary.chomp
      },
      :content => <<-EOF
#{summary}

Checkout the [Markdown Cheat Sheet](http://effectif.com/articles/markdown-cheat-sheet) to find out how to format your text to maximum effect.

If you want to add attachments to your pages you can drop them in the `#{Nesta::Config.attachment_path}` directory and refer to them using a URL such as [/attachments/my-file.png](/attachments/my-file.png) (`my-file.png` doesn't exist, but you get the idea). You can obviously refer to inline images using the same URL structure.
      EOF
    )
  end
end

namespace :article do  
  desc "Creates a new article"
  task :create do
    title = ask('Title?')
    speaker = ask('Speaker?')
    date = Date.today
    summary = ask('Summary?')
    if meeting_date = ask("Meeting Date (defaults to #{Date.today})? ") and meeting_date.length > 0
      begin
        meeting_date = Date.new(*meeting_date.split('-').map(&:to_i))
      rescue => err
        puts "Whoops, failed to process the date! The format must be #{Date.today}, you gave #{meeting_date}"
        raise err
        exit 1
      end
    else
      meeting_date = Date.today
    end
    
    factory = Factory.new
    filename = File.join("presentations", "#{speaker} #{title}".slugize)
    file = File.join(Nesta::Config.page_path, "#{filename}.mdown")
    location = "You can change this article by editing `#{file}`."
    factory.create_article(
      :heading => "#{speaker} on #{title}",
      :path => filename,
      :metadata => {
        "date" => date.to_s,
        "categories" => "presentations, #{speaker.slugize}",
        "read more" => "More details about the presentation",
        "summary" => summary.chomp,
        "thumbnail" => "#{speaker.slugize.gsub(/-/, '_')}_thumb.png"
      },
      :content => <<-EOF
Lorem ipsum ... blah blah.\n\n#{location}

Checkout the [Markdown Cheat Sheet](http://daringfireball.net/projects/markdown/syntax) to find out how to format your text to maximum effect.

If you want to add attachments to your articles you can drop them in the `#{Nesta::Config.attachment_path}` directory and refer to them using a URL such as [/attachments/my-file.png](/attachments/my-file.png) (`my-file.png` doesn't exist, but you get the idea). You can obviously refer to inline images using the same URL structure.

![#{speaker}](/attachments/#{speaker.slugize.gsub(/-/, '_')}_resized.jpg)

The next Agile Edmonton user group meeting will be on **Wednesday, #{meeting_date.to_s} at noon**.

#### Presentation Overview

Talk a bit about the presentation.

#### About the Speaker

Talk a bit about the speaker.

---

This meeting will be held in the [IBM Innovation Center at 10044-108th Street](http://maps.google.ca/maps?hl=en&safe=off&q=10044-108th+Street,edmonton,ab&ie=UTF8&hq=&hnear=10044+108+St+NW,+Edmonton,+Division+No.+11,+Alberta+T5J+3S7&gl=ca&ei=cJ9ZTLmPKNntnQev7_mxCQ&ved=0CBUQ8gEwAA&t=h&z=16). You can find more event information on our website at [agileedmonton.org](http://agileedmonton.org).

If possible, try to show up a bit before noon so that we can get started on time. Be sure to forward this information to anybody that would be interested.
      EOF
    )
    
    puts "Created article #{file}!"
  end
end

desc "Runs a server hosting your site on localhost:9393 using shotgun"
task :server do
  puts "Server launching on http://localhost:9393"
  system "bundle exec shotgun app.rb"
  puts "Bye!"
end

task :default => :"article:create"