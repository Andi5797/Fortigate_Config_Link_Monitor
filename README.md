# Fortigate_Config_Link_Monitor

## Cấu hình Link-Monitor trên FortiGate là cách để giám sát trạng thái kết nối mạng (thường là các đường WAN) bằng cách ping tới một IP đích. Nếu ping thất bại, FortiGate có thể tự động chuyển sang đường dự phòng hoặc thực hiện hành động như gỡ route, gửi cảnh báo, v.v.

### 🧭 Mục tiêu của Link Monitor
- Giám sát đường truyền (WAN1, WAN2…)
- Tự động failover khi mất kết nối
- Kết hợp với SD-WAN hoặc static route để đảm bảo uptime

### 🔧 Cấu hình chi tiết bằng CLI
#### 1. Tạo Link Monitor
```
config system link-monitor
    edit "WAN1-Monitor"
        set srcintf "wan1"                         # Interface cần giám sát
        set server "8.8.8.8" "1.1.1.1"              # IP đích để ping (có thể nhiều)
        set interval 5                              # Thời gian giữa các lần ping (giây)
        set timeout 1                               # Thời gian chờ phản hồi (giây)
        set failtime 3                              # Số lần ping thất bại liên tiếp để coi là mất kết nối
        set recoverytime 5                          # Số lần ping thành công liên tiếp để coi là đã phục hồi
        set update-cascade-interface disable        # Không cập nhật trạng thái interface vật lý
        set ha-priority 10                          # Ưu tiên HA nếu dùng
        set status enable
    next
end
```

📌 Giải thích:
- `srcintf`:                  cổng mạng cần giám sát (thường là WAN).
- `server`:                   IP đích để ping (nên chọn IP ổn định như DNS Google, Cloudflare).
- `interval`:                 khoảng thời gian giữa các lần ping.
- `failtime`:                 số lần thất bại liên tiếp để coi là “mất kết nối”.
- `recoverytime`:             số lần thành công liên tiếp để coi là “phục hồi”.
- `update-cascade-interface`: nếu bật, sẽ làm interface vật lý chuyển sang trạng thái down khi mất kết nối.

#### 2. Gắn vào static route để failover
**Ví dụ**: bạn có 2 đường WAN, WAN1 và WAN2. WAN1 là chính, WAN2 là dự phòng.
```
config router static
    edit 1
        set device "wan1"
        set gateway 192.168.1.1
        set distance 10
    next
    edit 2
        set device "wan2"
        set gateway 192.168.2.1
        set distance 20
    next
end
```

- Khi Link Monitor phát hiện WAN1 mất kết nối → route qua WAN1 bị gỡ → FortiGate tự động dùng WAN2.

#### 3. Kiểm tra trạng thái Link Monitor
`get system link-monitor`
hoặc:
`diagnose sys link-monitor status`
### 💡 Mẹo triển khai
- Nên dùng 2–3 IP đích để tránh false positive khi một IP tạm thời unreachable.
- Nếu dùng SD-WAN, Link Monitor sẽ tích hợp sẵn trong SD-WAN health check.
- Có thể cấu hình cảnh báo khi mất kết nối bằng email hoặc SNMP trap.

