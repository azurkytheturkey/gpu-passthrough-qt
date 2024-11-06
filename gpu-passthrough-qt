#!/usr/bin/env python3
import subprocess
import sys
import os
import re

from PyQt5.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QHBoxLayout,
    QPushButton, QLabel, QMessageBox, QScrollArea, QSizePolicy
)
from PyQt5.QtCore import Qt


class GPUDevice:
    def __init__(self, pci_id, description, vendor_id, device_id, driver, model_name):
        self.pci_id = pci_id
        self.description = description
        self.vendor_id = vendor_id
        self.device_id = device_id
        self.driver = driver
        self.model_name = model_name


def run_command(command):
    try:
        output = subprocess.check_output(
            command, stderr=subprocess.STDOUT, shell=True
        )
        return output.decode().strip()
    except subprocess.CalledProcessError as e:
        print(f"Error running command '{command}': {e.output.decode()}")
        return ""


def detect_devices():
    raw_output = run_command("lspci -nn | grep -E 'VGA|Audio'")
    devices = []
    if raw_output:
        for line in raw_output.splitlines():
            parts = line.split(" ")
            pci_id = parts[0]
            description = " ".join(parts[1:])
            match = re.search(r"\[(\w{4}):(\w{4})\]", line)
            if match:
                vendor_id, device_id = match.groups()
                driver = (
                    "amdgpu"
                    if "AMD" in description
                    else "nvidia"
                    if "NVIDIA" in description
                    else "nouveau"
                    if "NOUVEAU" in description
                    else "unknown"
                )

                # Extract model name
                if ":" in description:
                    description_after_colon = description.split(":", 1)[1].strip()
                else:
                    description_after_colon = description
                square_bracket_contents = re.findall(
                    r"\[([^\]]+)\]", description_after_colon
                )
                model_names = [
                    s
                    for s in square_bracket_contents
                    if s != f"{vendor_id}:{device_id}"
                    and not re.match(r"^\d+$", s)
                    and not re.match(r"^\d{4}$", s)
                ]
                if model_names:
                    model_name = model_names[-1]
                else:
                    model_name = description_after_colon.strip()

                devices.append(
                    GPUDevice(
                        pci_id,
                        description,
                        vendor_id,
                        device_id,
                        driver,
                        model_name,
                    )
                )
            else:
                print(f"Skipping line with unexpected format: {line}")

    paired_devices = []
    for gpu in devices:
        if "VGA" in gpu.description:
            for audio in devices:
                if (
                    "Audio" in audio.description
                    and gpu.pci_id[:2] == audio.pci_id[:2]
                    and gpu.vendor_id == audio.vendor_id
                ):
                    paired_devices.append((gpu, audio))
                    break
    return paired_devices


def write_vfio_conf(vga_id, audio_id, driver):
    vfio_conf_content = f"options vfio-pci ids={vga_id},{audio_id}\nsoftdep {driver} pre: vfio-pci\n"
    try:
        # Use sudo to write to the file
        cmd = f'echo "{vfio_conf_content}" | sudo tee /etc/modprobe.d/vfio.conf'
        subprocess.run(cmd, shell=True, check=True)
        print("Configuration written to /etc/modprobe.d/vfio.conf.")
    except subprocess.CalledProcessError:
        print("Error: Could not write to /etc/modprobe.d/vfio.conf.")


def update_initramfs():
    try:
        with open("/etc/os-release") as f:
            os_release = f.read().lower()
        if "arch" in os_release or "manjaro" in os_release:
            subprocess.run(["sudo", "mkinitcpio", "-P"], check=True)
        elif "endeavouros" in os_release:
            if os.path.isfile("/usr/bin/dracut"):
                subprocess.run(["sudo", "update_dracut_image"], check=True)
            else:
                subprocess.run(["sudo", "mkinitcpio", "-P"], check=True)
        elif "ubuntu" in os_release or "debian" in os_release:
            subprocess.run(["sudo", "update-initramfs", "-u"], check=True)
        elif "fedora" in os_release:
            subprocess.run(["sudo", "update_dracut_image"], check=True)
        else:
            print("OS not recognized. Please update initramfs manually.")
    except Exception as e:
        print(f"Error updating initramfs: {e}")


class GPUPassthroughManager(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("GPU Passthrough Manager")
        self.setGeometry(100, 100, 600, 600)
        self.paired_devices = detect_devices()

        main_vbox = QVBoxLayout()
        main_vbox.setSpacing(10)
        main_vbox.setContentsMargins(10, 10, 10, 10)

        # Scrollable area with a list of device buttons
        scrollable_area = QScrollArea()
        scrollable_area.setWidgetResizable(True)

        device_list_widget = QWidget()
        device_list_widget.setSizePolicy(QSizePolicy.Expanding, QSizePolicy.Minimum)
        device_layout = QVBoxLayout(device_list_widget)
        device_layout.setSpacing(10)  # Set distance between buttons
        device_layout.setAlignment(Qt.AlignTop)  # Align buttons to the top

        if not self.paired_devices:
            label = QLabel("No GPU-Audio pairs detected.")
            label.setAlignment(Qt.AlignCenter)
            label.setStyleSheet("font-size: 16pt;")
            device_layout.addWidget(label)
        else:
            for gpu, audio in self.paired_devices:
                button_label = (
                    f"{gpu.model_name} [{gpu.vendor_id}:{gpu.device_id}] "
                    f"[{audio.vendor_id}:{audio.device_id}]"
                )
                button = QPushButton(button_label)
                button.setFixedHeight(40)  # Same height as bottom buttons
                button.setSizePolicy(QSizePolicy.Expanding, QSizePolicy.Fixed)
                button.setStyleSheet("text-align: center;")
                button.clicked.connect(
                    lambda _, g=gpu, a=audio: self.on_device_button_clicked(g, a)
                )
                device_layout.addWidget(button)

        scrollable_area.setWidget(device_list_widget)
        main_vbox.addWidget(scrollable_area)

        # Button box
        button_hbox = QHBoxLayout()
        button_hbox.setSpacing(10)

        delete_button = QPushButton("Delete /etc/modprobe.d/vfio.conf")
        delete_button.setFixedHeight(40)
        delete_button.clicked.connect(self.on_delete_button_clicked)
        button_hbox.addWidget(delete_button)

        reboot_button = QPushButton("Reboot")
        reboot_button.setFixedHeight(40)
        reboot_button.clicked.connect(self.on_reboot_button_clicked)
        button_hbox.addWidget(reboot_button)

        main_vbox.addLayout(button_hbox)

        self.setLayout(main_vbox)

    def on_device_button_clicked(self, gpu, audio):
        write_vfio_conf(
            f"{gpu.vendor_id}:{gpu.device_id}",
            f"{audio.vendor_id}:{audio.device_id}",
            gpu.driver,
        )
        print(
            f"Configuration for {gpu.description} written to "
            "/etc/modprobe.d/vfio.conf."
        )
        update_initramfs()
        QMessageBox.information(
            self,
            "Configuration Written",
            "VFIO configuration has been written and initramfs updated. "
            "Please reboot to apply changes.",
        )

    def on_delete_button_clicked(self):
        vfio_conf_path = "/etc/modprobe.d/vfio.conf"
        if os.path.exists(vfio_conf_path):
            try:
                subprocess.run(f"sudo rm {vfio_conf_path}", shell=True, check=True)
                print(f"{vfio_conf_path} deleted.")
                update_initramfs()
                QMessageBox.information(
                    self,
                    "VFIO Configuration Deleted",
                    "VFIO configuration deleted. Initramfs updated. "
                    "Please reboot to apply changes.",
                )
            except subprocess.CalledProcessError:
                QMessageBox.warning(
                    self, "Error", f"Could not delete {vfio_conf_path}."
                )
        else:
            QMessageBox.warning(
                self, "File Not Found", f"{vfio_conf_path} does not exist."
            )

    def on_reboot_button_clicked(self):
        reply = QMessageBox.question(
            self,
            "Confirm Reboot",
            "Are you sure you want to reboot?",
            QMessageBox.Yes | QMessageBox.No,
            QMessageBox.No,
        )
        if reply == QMessageBox.Yes:
            try:
                subprocess.run(["sudo", "reboot"], check=True)
            except subprocess.CalledProcessError:
                QMessageBox.warning(
                    self, "Error", "Failed to reboot the system."
                )


def main():
    app = QApplication(sys.argv)
    window = GPUPassthroughManager()
    window.show()
    sys.exit(app.exec_())


if __name__ == "__main__":
    main()
