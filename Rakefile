require "rake/contrib/ftptools"
require "rake/clean"

module Rake
  class FtpUploader  
    # Deletes all files in a folder like cache/management. Does
    # not delete folders
    def delete_files(folder)
      delete_files_recursive(folder)
    end

    private 
    # starts with a folder, like cache/management and deletes
    # all files recursively BUT not the folders.
    def delete_files_recursive(file_or_folder)
      folder = true
      begin
        @ftp.chdir(file_or_folder)
      rescue
        # this is a file, no folder => delete
        folder = false
      end

      if folder
        file_list = @ftp.nlst('*')
        file_list.each { |f| delete_files_recursive(f) }
        @ftp.chdir('..')
      else
        puts "Delete #{file_or_folder}" if @verbose
        delete file_or_folder
      end
    end

    # deletes a single file
    def delete(file)
      puts "Deleting #{file}" if @verbose
      begin
        @ftp.delete(file)
      rescue
        puts "Could not delete file #{file}"
      end
    end

  end
end

CLEAN.include('./_site/assets/**/*')
CLOBBER.include('./_site/')

namespace :assets do
  desc 'Builds jekyll site and compiles assets'
  task :precompile do
    puts `bundle exec jekyll clean && JEKYLL_ENV=production bundle exec jekyll build && echo $JEKYLL_ENV`
  end
end

desc 'Upload site via FTPS'
task :upload do
 Dir.chdir('./_site/')
 Rake::FtpUploader.connect('/','example.com','user', 'password') do |ftp|
    ftp.verbose = true
    ftp.upload_files('**/*')
  end 
end

desc 'Build, deploy and upload site'
task :deploy => ['assets:precompile'] do 
    Rake::Task[:upload].invoke
end

namespace :cache do
  desc 'Cleans remote assets folder'
  task :clean do
    Rake::FtpUploader.connect('/','example.com','user', 'password') do |ftp|
      ftp.verbose = true
      ftp.delete_files('./assets')
    end
  end
end

task :default do
  puts `bundle exec jekyll build`
end
