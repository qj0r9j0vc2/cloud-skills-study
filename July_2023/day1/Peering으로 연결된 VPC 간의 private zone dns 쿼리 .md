# Peering으로 연결된 VPC 간의 private zone dns 쿼리 

# Create Peer VPC 
![](<./image/스크린샷 2023-07-31 오전 2.02.42.png>)
기존 VPC와 CIDR 범위 다르게 지정함.

# Peering
![](<./image/스크린샷 2023-07-31 오전 2.06.40.png>)


# Route table
![](<./image/스크린샷 2023-07-31 오전 2.07.51.png>)
나머지 반대 쪽 rt도 구성해준다.

# DNS 설정
![](<./image/스크린샷 2023-07-31 오전 2.10.12.png>)


# Test
![](<./image/스크린샷 2023-07-31 오전 2.11.36.png>)

![](<./image/스크린샷 2023-07-31 오전 2.12.23.png>)
private DNS 복붙 dig -> 잘 나옴.