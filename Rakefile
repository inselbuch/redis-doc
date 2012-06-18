task :default => [:parse, :spellcheck]

task :parse do
  require "json"
  require "batch"
  require "rdiscount"

  Batch.each(Dir["**/*.json"] + Dir["**/*.md"]) do |file|
    if File.extname(file) == ".md"
      RDiscount.new(File.read(file)).to_html
    else
      JSON.parse(File.read(file))
    end
  end
end

task :spellcheck do
  require "json"

  `mkdir -p tmp`

  IO.popen("aspell --lang=en create master ./tmp/dict", "w") do |io|
    io.puts(JSON.parse(File.read("commands.json")).keys.map(&:split).flatten.join("\n"))
    io.puts(File.read("wordlist"))
  end

  Dir["**/*.md"].each do |file|
    command = %q{
      ruby -pe 'gsub /^    .*$/, ""' |
      ruby -pe 'gsub /`[^`]+`/, ""' |
      ruby -e 'puts $stdin.read.gsub /\[([^\]]+)\]\(([^\)]+)\)/m, "\\1"' |
      aspell -H -a --extra-dicts=./tmp/dict 2>/dev/null
    }

    words = `cat '#{file}' | #{command}`.lines.map do |line|
      line[/^& ([^ ]+)/, 1]
    end.compact

    puts "#{file}: #{words.uniq.sort.join(" ")}" if words.any?
  end
end

namespace :format do

  def format(file)
    return unless File.exist?(file)

    STDOUT.print "formatting #{file}..."
    STDOUT.flush

    matcher = /^(?:\A|\r?\n)((?:[a-zA-Z@].+?\r?\n)+)/m
    body = File.read(file).gsub(matcher) do |match|
      formatted = nil

      IO.popen("par p0s0w80", "r+") do |io|
        io.puts match
        io.close_write
        formatted = io.read
      end

      formatted
    end

    File.open(file, "w") do |f|
      f.print body
    end

    STDOUT.puts
  end

  task :file, :path do |t, args|
    format(args[:path])
  end

  task :all do
    Dir["commands/*.md"].each do |path|
      format(path)
    end
  end
end
