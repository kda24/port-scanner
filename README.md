# port-scanner
import socket
import threading

open_ports = []
lock = threading.Lock()


def grab_banner(host, port):
    try:
        s = socket.socket()
        s.settimeout(2)
        s.connect((host, port))

        try:
            banner = s.recv(1024).decode('utf-8', errors='ignore').strip()
        except:
            banner = "no banner"

        s.close()

        return banner if banner else "no banner"

    except:
        return "no banner"


def scan_port(host, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)

    result = s.connect_ex((host, port))
    s.close()

    if result == 0:
        with lock:
            open_ports.append(port)


host = input("Enter target IP (or press Enter for 127.0.0.1): ").strip()

if host == "":
    host = "127.0.0.1"

print(f"\nScanning {host} ...\n")

threads = []

for port in range(1, 1024):
    t = threading.Thread(target=scan_port, args=(host, port))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

with open("results.txt", "w") as f:
    f.write(f"Scan results for: {host}\n")
    f.write("=" * 40 + "\n")

    if open_ports:
        for port in sorted(open_ports):
            banner = grab_banner(host, port)

            line = f"[OPEN] Port {port} → {banner}"

            print(line)
            f.write(line + "\n")

    else:
        print("No open ports found.")
        f.write("No open ports found.\n")

print("\nResults saved to results.txt")
print("Scan complete.")
