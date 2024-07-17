# Location-tracker

This project demonstrates how to visualize network traffic using Python, Wireshark, and Google Maps. We will take a file of network traffic and convert it into a visual presentation using Google Maps.


## Prerequisites

Besides the Python code  you will need several external resources and applications:

### GeoLiteCity
Download the GeoLiteCity database to translate IP addresses into geographical locations (longitude & latitude). [Download GeoLiteCity]

### Wireshark
Wireshark is used to capture network traffic on your device. The captured traffic will be the input for our Python script. [Download Wireshark](https://www.wireshark.org/).

For an introduction to Wireshark, check out this playlist on Vinsloev Academy: [Wireshark Basics]

### Python 3
Ensure Python 3.x is installed on your device. [Download Python](https://www.python.org/).

## Capturing Network Traffic

1. Open Wireshark and select an interface with traffic.
2. Wireshark starts capturing traffic immediately.
3. Stop the capture and save the data in pcap format: File -> Export Specified Packets -> pcap format.

## Python Implementation

### Importing Libraries
```python
import dpkt
import socket
import pygeoip

gi = pygeoip.GeoIP('GeoLiteCity.dat')
```

### Main Method
```python
def main():
    f = open('wire.pcap', 'rb')
    pcap = dpkt.pcap.Reader(f)
    kmlheader = '<?xml version="1.0" encoding="UTF-8"?> \n<kml xmlns="http://www.opengis.net/kml/2.2">\n<Document>\n'\
    '<Style id="transBluePoly">' \
                '<LineStyle>' \
                '<width>1.5</width>' \
                '<color>501400E6</color>' \
                '</LineStyle>' \
                '</Style>'
    kmlfooter = '</Document>\n</kml>\n'
    kmldoc=kmlheader+plotIPs(pcap)+kmlfooter
    print(kmldoc)
```

### Extracting IPs
```python
def plotIPs(pcap):
    kmlPts = ''
    for (ts, buf) in pcap:
        try:
            eth = dpkt.ethernet.Ethernet(buf)
            ip = eth.data
            src = socket.inet_ntoa(ip.src)
            dst = socket.inet_ntoa(ip.dst)
            KML = retKML(dst, src)
            kmlPts = kmlPts + KML
        except:
            pass
    return kmlPts
```

### Attaching Geo Location
```python
def retKML(dstip, srcip):
    dst = gi.record_by_name(dstip)
    src = gi.record_by_name('x.xxx.xxx.xxx')
    try:
        dstlongitude = dst['longitude']
        dstlatitude = dst['latitude']
        srclongitude = src['longitude']
        srclatitude = src['latitude']
        kml = (
            '<Placemark>\n'
            '<name>%s</name>\n'
            '<extrude>1</extrude>\n'
            '<tessellate>1</tessellate>\n'
            '<styleUrl>#transBluePoly</styleUrl>\n'
            '<LineString>\n'
            '<coordinates>%6f,%6f\n%6f,%6f</coordinates>\n'
            '</LineString>\n'
            '</Placemark>\n'
        )%(dstip, dstlongitude, dstlatitude, srclongitude, srclatitude)
        return kml
    except:
        return ''
```

### Running the Script
```python
if __name__ == '__main__':
    main()
```

## Adding Data to Google Maps

1. Copy the generated KML data into a file with a .kml extension.
2. Navigate to [Google My Maps](https://www.google.com/mymaps).
3. Create a new map and import the .kml file.
4. View the network traffic displayed on the map.
