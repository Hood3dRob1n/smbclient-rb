smbclient-rb
============

A Ruby class which wraps the smbclient tool to make easily available from your Ruby scripts/apps

Basic Connection with Clear Text Credentials:
=================================================
client = RubySmbClient.new('192.168.2.16', 139, 'c$', 'Administrator', 'P@ssw0rd1', nil, false)
if client.can_we_connect?
  puts "Connected with valid user & clear-text password!"
  puts "OS: #{$os}"
  output = client.list_shares_credentialed
  if not output.nil? and output != ''
    count=0
    output.each do |line|
      if line =~ /Server\s+Comment/
        puts
        puts line.chomp
      else
        if count.to_i == 0
          puts "Available Shares: "
          count = count.to_i + 1
        else
          puts line.chomp
        end
      end
    end
  end
else
  puts "Failed to Connect!"
  puts "OS: #{$os}"
end



Basic Connection with Hashed Credentials:
=================================================
client = RubySmbClient.new('192.168.2.16', 139, 'c$', 'Administrator', 'ae974876d974abd805a989ebead86846', nil, true)
if client.can_we_connect?
  puts "Connected with valid user & hashed password!"
  puts "OS: #{$os}"
  output = client.list_shares_credentialed
  if not output.nil? and output != ''
    count=0
    output.each do |line|
      if line =~ /Server\s+Comment/
        puts
        puts line.chomp
      else
        if count.to_i == 0
          puts "Available Shares: "
          count = count.to_i + 1
        else
          puts line.chomp
        end
      end
    end
  end
else
  puts "Failed to Connect!"
  puts "OS: #{$os}"
end



Run some Single one-off smbclient commands on our own:
=================================================
output = client.smb_cmd('ls')
if not output.nil?
  count=0
  output.each do |line|
    if count.to_i == 0
      puts "Directory Listing for: \\\\\\\\#{client.client_creds[1]}:#{client.client_creds[2]}\\\\#{client.client_creds[3]}"
      count = count.to_i + 1
    else
      puts line.chomp
    end
  end
else
  puts "Command did not return any output!"
end



Drop to SMB Shell:
=================================================
puts "Dropping to SMB Shell...."
client.smb_shell
puts



Download a file over SMB:
=================================================
rfile='bootz.ini' # Bad File Name
lfile='/root/Desktop/downloaded_over_smb-boot.ini.txt'
puts "Downloading: #{rfile}"
if client.smb_download(rfile, lfile)
  if File.exists?(lfile)
    puts "Successfully downloaded #{rfile} to #{lfile}!"
  else
    puts "File should have been downloaded, I dont know what happened......?"
  end
end
puts

rfile='boot.ini' # Good File Name
puts "Downloading: #{rfile}"
if client.smb_download(rfile, lfile)
  if File.exists?(lfile)
    puts "Successfully downloaded #{rfile} to #{lfile}!"
  else
    puts "File should have been downloaded, I dont know what happened......?"
  end
end
puts



Download & Read file from server:
=================================================
puts "File: #{rfile}"
puts "Content: "
if client.smb_download(rfile, rfile)
  system("cat #{rfile}")
end




Upload a local file to server & then rename it:
=================================================
# upload phpinfo.php to web dir (C:\wamp\www\) on remote server, then rename to pinfo.php
lfile='/root/Desktop/phpinfo.php'
rdir="wamp\\\\www\\\\"
# The upload option looks for the file in current directory
# Causes issues when pulling from elsewhere unless file path re-written....
Dir.chdir(lfile.split('/')[0..-2].join('/')) do 
  if client.smb_upload(lfile.split('/')[-1], rdir)
    puts "File uploaded to remote server!"
    puts "Attempting to rename remote file now....."
    if client.smb_file_rename('phpinfo.php', 'pinfo.php', 'wamp\\\\www')
      puts "File successfully renamed!"
    else
      puts "Problem renaming remote file!"
    end
  end
end



1) Making a directory on remote server
2) Upload a file, list directory content
3) Then remove file, list, remove dir and list again
=================================================
puts "Trying to make a new directory....."
#if client.smb_mkdir('smbtest', 'wamp\\\\wwww')
if client.smb_mkdir('smbtest', 'wamp\\\\www')
#  if client.smb_upload('smb-test.txt', 'wamp\\\\www\\\\smbtest')
  if client.smb_upload('smb-test.txt', 'wamp\\\\www\\\\smbtest')
    puts "Created new Directory & Uploaded test file!"
    output = client.smb_cmd('ls', 'wamp\\\\www')
    puts output
    output = client.smb_cmd('ls', 'wamp\\\\www\\\\smbtest')
    puts output
    if client.smb_rm('smb-test.txt', 'wamp\\\\www\\\\smbtest')
      puts "Deleted smb-test.txt file!"
      output = client.smb_cmd('ls', 'wamp\\\\www\\\\smbtest')
      puts output
      if client.smb_rmdir('smbtest', 'wamp\\\\www')
        puts "Deleted the smbtest directory!"
        output = client.smb_cmd('ls', 'wamp\\\\www')
        puts output
      else
        puts "Unable to delete smbtest directory!"
      end
    else
      puts "Unable to delete smb-test.txt file!"
    end
  else
    puts "Problem uploading test file...."
  end
else
  puts "Problem making new directory, sorry....."
end




Download all files from directory on remote server:
=================================================
puts "Downloading the Web Directory from remote server......"
output= client.download_dir('www', 'wamp', confirmation=false)
if not output.nil?
  output.each do |line|
    if line =~ /getting file .+ of size \d+ as (.+) \(/i
      puts "Downloaded: #{$1}"
    end
  end
  puts
  puts "Downloads Successfull!"
end



Perform a SMB OS Version Scan:
=================================================
puts "Running SMB OS Discovery Scan....."
(1..254).each do |num|
  $os=''
  client = RubySmbClient.new("192.168.1.#{num}") # Edit IP Range to suit your need
  shares = client.list_shares_anonymous
  msg = shares.join.to_s.sub('false', '').sub('true', '')
  if not $os.nil? and $os != ''
    puts "[*] Host: 192.168.1.#{num}, OS: #{$os}, DOMAIN: #{$domain}"
    puts "  #{msg}"
  else
    puts "[*] Host: 192.168.1.#{num}"
  end
end

