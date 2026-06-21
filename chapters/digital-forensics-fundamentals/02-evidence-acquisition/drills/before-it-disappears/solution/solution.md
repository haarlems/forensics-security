# Solution

``` bash
# we analyze the files
$ file core.12003
core.12003: ELF 64-bit LSB core file, x86-64, version 1 (SYSV), SVR4-style, from 'systemd-resolved./systemd-resolved', real uid: 1000, effective uid: 1000, real gid: 1000, effective gid: 1000, execfn: './systemd-resolved', platform: 'x86_64'
$ file systemd-resolved
systemd-resolved: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c3fa1eced36a430aadc32539127595fd0d656b66, for GNU/Linux 3.2.0, with debug_info, not stripped
# strings reveals a fake flag
$ strings core.12003
0 Igp
[..]
SSS{n0t_th3_r34l_0ne_k3ep_d1ggiNg}
# we load both artifacts into a gdb
$ gdb ./systemd-resolved core.12003
# we find some interesting variables
(gdb) info variables
All defined variables:
[..]
File chall1.c:
61:     struct beacon_blob *g_blob;
44:     static const struct beacon_config g_template[27];
[..]
# we check the structures
(gdb) ptype struct beacon_blob
type = struct beacon_blob {
    char magic[8];
    uint32_t count;
    uint32_t reserved;
    struct beacon_config configs[27];
}
(gdb) ptype struct beacon_config
type = struct beacon_config {
    uint32_t jitter;
    char c;
    uint32_t sleep_ms;
    char pad[3];
}
# we follow the pointers and we start seeing the flag
(gdb) p* g_blob
$1 = {magic = "BEACONv1", count = 27, reserved = 0, configs = {{jitter = 2593, c = 83 'S',
      sleep_ms = 30000, pad = "\000\000"}, {jitter = 8004, c = 83 'S', sleep_ms = 28500,
      pad = "\000\000"}, {jitter = 2099, c = 83 'S', sleep_ms = 31000, pad = "\000\000"}, {
      jitter = 11280, c = 123 '{', sleep_ms = 27750, pad = "\000\000"}, {jitter = 4425, c = 109 'm',
      sleep_ms = 29900, pad = "\000\000"}, {jitter = 15106, c = 48 '0', sleep_ms = 26100,
      pad = "\000\000"}, {jitter = 1303, c = 115 's', sleep_ms = 32400, pad = "\000\000"}, {
      jitter = 10670, c = 84 'T', sleep_ms = 28850, pad = "\000\000"}, {jitter = 7267, c = 95 '_',
      sleep_ms = 30150, pad = "\000\000"}, {jitter = 2033, c = 116 't', sleep_ms = 27300,
      pad = "\000\000"}, {jitter = 8768, c = 48 '0', sleep_ms = 31600, pad = "\000\000"}, {
      jitter = 3740, c = 95 '_', sleep_ms = 29050, pad = "\000\000"}, {jitter = 13056, c = 76 'L',
      sleep_ms = 26900, pad = "\000\000"}, {jitter = 6741, c = 101 'e', sleep_ms = 30700,
      pad = "\000\000"}, {jitter = 3112, c = 65 'A', sleep_ms = 28200, pad = "\000\000"}, {
      jitter = 9985, c = 53 '5', sleep_ms = 31950, pad = "\000\000"}, {jitter = 4918, c = 116 't',
      sleep_ms = 27050, pad = "\000\000"}, {jitter = 15890, c = 95 '_', sleep_ms = 29550,
      pad = "\000\000"}, {jitter = 2338, c = 118 'v', sleep_ms = 30900, pad = "\000\000"}, {
      jitter = 11696, c = 111 'o', sleep_ms = 26450, pad = "\000\000"}, {jitter = 6148, c = 49 '1',
      sleep_ms = 32100, pad = "\000\000"}, {jitter = 1121, c = 65 'A', sleep_ms = 28650,
      pad = "\000\000"}, {jitter = 12656, c = 116 't', sleep_ms = 30300, pad = "\000\000"}, {
      jitter = 7725, c = 105 'i', sleep_ms = 27600, pad = "\000\000"}, {jitter = 2904, c = 76 'L',
      sleep_ms = 31250, pad = "\000\000"}, {jitter = 9481, c = 51 '3', sleep_ms = 29200,
      pad = "\000\000"}, {jitter = 5697, c = 125 '}', sleep_ms = 28000, pad = "\000\000"}}}
# we can loop to extract it
(gdb) while $i < g_blob->count
 >printf "%c",g_blob->configs[$i].c
 >set $i = $i + 1
 >end
SSS{m0sT_t0_LeA5t_vo1AtiL3}
```
