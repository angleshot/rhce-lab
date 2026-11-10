# -*- mode: ruby -*-
# vi: set ft=ruby :
# =============================================================
# RHCE 랩 배포용: control 1 + node 5 (VirtualBox)
# 필요 박스: control, node (vagrant box add로 등록)
# 사용법: vagrant up / vagrant halt / vagrant destroy -f
# =============================================================

Vagrant.configure("2") do |config|
  config.vm.boot_timeout = 600
  config.ssh.insert_key = false   # 부팅 키 자동교체 끔 — admin/admin 로그인 유지
  config.vm.synced_folder '.', '/vagrant', disabled: true  # Guest Additions 없음 → 공유폴더 비활성화

  # ---- control ----
  config.vm.define "control" do |c|
    c.vm.box = "control"
    c.vm.hostname = "control.example.com"
    c.vm.network "private_network", ip: "172.17.29.10"
    c.vm.provider "virtualbox" do |vb|
      vb.memory = 4096
      vb.cpus = 2
      vb.name = "rhce-control"
      vb.customize ["modifyvm", :id, "--groups", "/rhce-lab"]  # VirtualBox Manager에서 rhce-lab 그룹으로 묶임
      vb.gui = true   # Server with GUI: 부팅화면/GNOME 창 표시 (SSH만 쓰려면 false)
    end
  end

  # ---- node1 ~ node5 ----
  # 연습용 디스크: node1~3 = 2GB, node4~5 = 1GB
  NODE_DISK = { "node1" => 2, "node2" => 2, "node3" => 2, "node4" => 1, "node5" => 1 }

  (1..5).each do |i|
    name = "node#{i}"
    config.vm.define name do |n|
      n.vm.box = "node"
      n.vm.hostname = "#{name}.example.com"
      n.vm.network "private_network", ip: "172.17.29.1#{i}"
      n.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 1
        vb.name = "rhce-#{name}"
        vb.customize ["modifyvm", :id, "--groups", "/rhce-lab"]  # VirtualBox Manager에서 rhce-lab 그룹으로 묶임
        vb.gui = true    # headless 크래시(0xc0000005) 방지 — SentinelOne 없는 PC는 false로 바꿔도 됨
        disk_dir = File.join(Dir.pwd, "disks")
        Dir.mkdir(disk_dir) unless Dir.exist?(disk_dir)
        disk = File.expand_path("disk_#{name}.vdi", disk_dir)
        unless File.exist?(disk)
          vb.customize ["createhd", "--filename", disk, "--size", NODE_DISK[name] * 1024]
        end
        vb.customize ["storageattach", :id, "--storagectl", "SATA",
                      "--port", 1, "--device", 0, "--type", "hdd", "--medium", disk]
      end
    end
  end
end
