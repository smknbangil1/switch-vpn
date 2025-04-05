# Switch On-Off OpenVPN Client di mikrotik
Berikut ini adalah skrip Mikrotik untuk **memantau koneksi VPN client** (OpenVPN interface bernama `MyDNS`) dan **melakukan auto-reconnect** (disable lalu enable) jika terjadi **kegagalan ping 5x berturut-turut ke IP `10.8.8.1`**:

---

### ✅ **Skrip Mikrotik Revisi (tanpa `:break`)**
```mikrotik
# Skrip: check-vpn
:local vpnInterface "MyDNS"
:local targetIP "10.8.8.1"
:local failCount 0
:local maxFail 5
:local success false

:for i from=1 to=$maxFail do={
    :if ([/ping $targetIP count=1 interval=1s] = 0) do={
        :set failCount ($failCount + 1)
    } else={
        :set success true
    }
    :delay 1
}

:if (!$success && $failCount = $maxFail) do={
    :log warning "VPN interface $vpnInterface gagal ping $maxFail kali. Restarting interface..."
    /interface ovpn-client disable $vpnInterface
    :delay 3
    /interface ovpn-client enable $vpnInterface
}
```

---

### 🔁 Cara Kerja Skrip:
- Skrip ping sebanyak 5 kali ke `10.8.8.1`.
- Jika semuanya gagal (`failCount = 5` dan tidak ada satupun yang sukses), maka VPN interface `MyDNS` akan di-*restart* (disable lalu enable).
- Tidak pakai `:break`, tapi memakai flag `success` sebagai penanda jika ada 1 ping sukses.

---

### 🕒 Penjadwalan di Scheduler
```bash
/system scheduler add name=check-vpn interval=1m on-event="/system script run check-vpn" start-time=startup
```

---

Kalau butuh tambahan fitur seperti mencatat waktu terakhir reconnect atau jumlah reconnect harian, bisa juga kita tambahkan. Mau?
