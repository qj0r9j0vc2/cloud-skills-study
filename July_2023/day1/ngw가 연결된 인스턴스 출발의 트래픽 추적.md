# ngw가 연결된 인스턴스 출발의 트래픽 추적

- webserver: 80포트에서 service apache2 start
- private-ec2-test: ngw가 연결된 private subnet에서 public IP 자동 할당 활성화 instance

# webserver

---

```yaml
$ sudo tcpdump -A -vv -nn port 80 -w devops-port-80-2023-07-31:01:29.pcap
```

# private-ec2-test

---

```yaml
$ curl {{webserver-ip}}
```

# webserver
![](<./image/스크린샷 2023-07-31 오전 1.46.37.png>)
![](<./image/스크린샷 2023-07-31 오전 1.47.24.png>)
![](<./image/스크린샷 2023-07-31 오전 1.47.37.png>)