# ArborPI_project
ArborPI team of Astro PI programm
from orbit import ISS
from picamera import PiCamera
from pathlib import Path
from time import sleep
from datetime import datetime, timedelta

start_time = datetime.now()

def convert(angle):
    """
    Convert a `skyfield` Angle to an EXIF-appropriate
    representation (rationals)
    e.g. 98° 34' 58.7 to "98/1,34/1,587/10"

    Return a tuple containing a boolean and the converted angle,
    with the boolean indicating if the angle is negative.
    """
    sign, degrees, minutes, seconds = angle.signed_dms()
    exif_angle = f'{degrees:.0f}/1,{minutes:.0f}/1,{seconds*10:.0f}/10'
    return sign < 0, exif_angle

def capture(camera, image):
    """Use `camera` to capture an `image` file with lat/long EXIF data."""
    point = ISS.coordinates()

    coordinate_pair = (
        point.latitude.degrees,
        point.longitude.degrees)

    elevationList = reverse_geocoder.search(coordinate_pair, mode=1)
    townList = reverse_geocoder.search(coordinate_pair, mode=1)

    elevation[15] = elevationList
    town[1] = townList

    # Convert the latitude and longitude to EXIF-appropriate representations
    south, exif_latitude = convert(point.latitude)
    west, exif_longitude = convert(point.longitude)

    camera.exif_tags['GPS.GPSLatitude'] = exif_latitude
    camera.exif_tags['GPS.GPSLatitudeRef'] = "S" if south else "N"
    camera.exif_tags['GPS.GPSLongitude'] = exif_longitude
    camera.exif_tags['GPS.GPSLongitudeRef'] = "W" if west else "E"
    camera.exif_tags['elevation'] = elevation
    camera.exif_tags['nearestTown'] = town


    # Capture the image
    camera.capture(image)

base_folder = Path(__file__).parent.resolve()

cam = PiCamera()
cam.resolution = (1296,972)
camera.start_preview()
sleep(2)

now_time = datetime.now()

while (now_time < start_time + timedelta(minutes=175)):
    for filename in capture(cam, f"{base_folder}/image_{counter:03d}.jpg"):
        print(f'Captured {filename}')
        sleep(21) # wait 5 minutes
        now_time = datetime.now()




