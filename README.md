pip install ipaddress
import socket
import ipaddress
from concurrent.futures import ThreadPoolExecutor

def scan_port(ip, port):
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.settimeout(1)
            s.connect((ip, port))
            return port, True
    except (socket.timeout, ConnectionRefusedError):
        return port, False

def scan_device(ip, ports):
    open_ports = []
    for port in ports:
        result = scan_port(ip, port)
        if result[1]:
            open_ports.append(result[0])
    return ip, open_ports

def main():
    target_network = input("Enter the target network (e.g., 192.168.1.0/24): ")
    ports = range(1, 1025)  # Common ports range from 1 to 1024

    network = ipaddress.ip_network(target_network, strict=False)
    devices = []

    with ThreadPoolExecutor(max_workers=50) as executor:
        future_to_ip = {executor.submit(scan_device, str(ip), ports): str(ip) for ip in network.hosts()}

        for future in concurrent.futures.as_completed(future_to_ip):
            ip = future_to_ip[future]
            open_ports = future.result()
            if open_ports:
                devices.append((ip, open_ports))

    for device in devices:
        print(f"Device {device[0]} has open ports: {', '.join(map(str, device[1]))}")

if __name__ == "__main__":
    main()
