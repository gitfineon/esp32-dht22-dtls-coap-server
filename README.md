# esp32-dht22-dtls-coap-server

Based on the [esp-idf](https://github.com/espressif/esp-idf) examples for the DTLS-enabled CoAP server and the [DHT22 example](https://github.com/gosouth/DHT22) from Ricardo Timmermann.

## Set up the ESP32 node

If you are not using a Linux machine please consider the docs of the projects in use. All steps described here are done with Ubuntu 18.04 but should be possible on a Debian/Raspian system, too.

### Toolchain for ESP32

[Linux Prerequisites](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/linux-setup.html) on Debian based system: 

`sudo apt-get install git wget libncurses-dev flex bison gperf python python-pip python-setuptools python-serial python-click python-cryptography python-future python-pyparsing python-pyelftools cmake ninja-build ccache`

Clone the repository including its submodules
`git clone  --recurse-submodules https://github.com/gitfineon/esp32-dht22-dtls-coap-server.git`

`cd esp32-dht22-dtls-coap-server/esp-idf`

The following script installs the toolchain to the users home-directory by default (`$HOME/.espressif`). Set the environment variable `IDF_TOOLS_PATH` if you like to choose another destination.

`./install.sh`

Execute the export script to use the tools within this terminal.

> The script is not compatible with zsh, so you have to use bash.

`./export.sh`

To make the tools available system-wide add this script with its absolute path to your .profile or another shell config file.

### Configure, build and flash the ESP32

Now switch to to the project folder

`cd ../coap_server_dht22` and enter the configuration assistant

`idf.py menuconfig`

```
Serial flasher config 
  -> Flash SPI speed (80 MHz)
  -> Flash size (4 MB)
  -> monitor baud rate (115200 bps)

Example Connection Configuration
  -> (YourSSID) WiFi SSID
  -> (CorrespondingPass) Wifi Password

Component config
  -> CoAP Encryption mehtod (PKI Certificates)
```

> If you want to connect to the gateway, use the gateway's Wifi credentials.

`idf.py build` can be used to build the project

Finally, if your ESP32 device is connected via /dev/ttyUSB0, `idf.py flash monitor` can be used to flash the device and directly start a serial console to monitor the device. Use '-p PORT' option to set a specific serial port.

If code changed, the flash target automatically rebuilds.

Exit the monitor by pressing `Ctrl+]`

## Test communication using coap-client from libcoap

Install [libcoap](https://libcoap.net/install.html) to make use of their DTLS-enabled terminal app coap-client.

Make sure you are using the `--enable-dtls` flag and specific DTLS library. I have been using the `--with-gnutls` flag after installing gnutls with the package-manager:

```
sudo apt install gnutls-bin
cd libcoap
./autogen.sh
./configure --enable-dtls --with-gnutls
make
make install
```

For this example we use the certificate delivered with the example and store their location in a variable:

`export ESP32_PKI_CERTS="[absolute-path-to]/esp32-dht22-dtls-coap-server/coap_server_dht22/main/certs"`

The node is configured to use DHCP, so we have to find out which IP address the device got, for instance using the android app 'fing' or the router page with all connected network devices.

If the device got the IP 192.168.2.109

`coap-client -R ${ESP32_PKI_CERTS} coap_server_dht22/main/certs -m get coaps://192.168.2.109/description`

This prints a warning, because the certificate is not signed by a known CA, but the communication is secured via DTLS.

To receive the values of the DHT22 sensor use the resources `temperature` and `humidity` instead of `desctiption`. Both sensor resources are configured as read-only (get), while the description is adjustable with put and clear.

## Known issues and limitations

* The node is not secured against flodding, which might lead to denial of service attacks
* Messages are sometimes discarded with a warning about 'invalid epoch'
* Sometimes warning about received packet with illegal length
* Sometimes the client does not print any received value
* Performance is quite slow, and the delays in the RTOS are not optimized.
