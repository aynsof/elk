Vagrant.configure("2") do |config|
  config.vm.box = "trusty"
  config.ssh.username = "vagrant"
  config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"

  config.vm.network :forwarded_port, guest: 9200, host: 9200
  config.vm.network :forwarded_port, guest: 9300, host: 9300
  config.vm.network :forwarded_port, guest: 5601, host: 5601
  config.vm.network :forwarded_port, guest: 80, host: 8090

$install = <<SCRIPT
#sudo add-apt-repository -y ppa:webupd8team/java
sudo apt-get update
#sudo apt-get -y install oracle-java7-installer
sudo apt-get -y install openjdk-7-jre
wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
echo 'deb http://packages.elasticsearch.org/elasticsearch/1.1/debian stable main' | sudo tee /etc/apt/sources.list.d/elasticsearch.list
sudo apt-get update
sudo apt-get -y install elasticsearch
sudo echo 'script.disable_dynamic: true' | sudo tee /etc/elasticsearch/elasticsearch.yml
sudo service elasticsearch restart
sudo update-rc.d elasticsearch defaults 95 10
echo Elasticsearch installed!

cd ~; wget https://download.elasticsearch.org/kibana/kibana/kibana-3.0.1.tar.gz
tar xvf kibana-3.0.1.tar.gz
sed -i.bak 's#elasticsearch: "http://"+window.location.hostname+":9200",#elasticsearch: "http://192.168.21.238:9200",#g' ~/kibana-3.0.1/config.js
sudo mkdir -p /var/www/kibana3
sudo cp -R ~/kibana-3.0.1/* /var/www/kibana3/
echo Kibana installed!

sudo apt-get install nginx -y --force-yes
cd ~; wget https://gist.githubusercontent.com/thisismitch/2205786838a6a5d61f55/raw/f91e06198a7c455925f6e3099e3ea7c186d0b263/nginx.conf
sed -i.bak 's#root  /usr/share/kibana3;#root /var/www/kibana3;#g' nginx.conf
sudo cp nginx.conf /etc/nginx/sites-available/default
sudo service nginx restart
echo Nginx installed!

echo 'deb http://packages.elasticsearch.org/logstash/1.4/debian stable main' | sudo tee /etc/apt/sources.list.d/logstash.list
sudo apt-get update
sudo apt-get install logstash=1.4.2-1-2c0f5a1 -y --force-yes
echo 'input {
  lumberjack {
    port => 5601
    type => "logs"
  }
}' | sudo tee /etc/logstash/conf.d/01-lumberjack-input.conf
echo 'filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}' | sudo tee /etc/logstash/conf.d/10-syslog.conf
echo 'output {
  elasticsearch { host => localhost }
  stdout { codec => rubydebug }
}' | sudo tee /etc/logstash/conf.d/30-lumberjack-output.conf
sudo service logstash restart
echo Logstash installed!

SCRIPT

  config.vm.provision :shell, :inline => $install, :privileged => false

end

