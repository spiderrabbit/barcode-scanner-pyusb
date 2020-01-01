# barcode-scanner-pyusb
#Sample script for pyusb to grab barcode scanner USB keyboard raw data and turn into useful string

#Most barcode scanners deliver their data by default via HID keyboard device. The script uses pyusb to grab the keyboard device so the data can be used by Python elsewhere.

#Note you'll need to change the hex USB device identifier string (get via lsusb), and you may need to alter the code which indicates string termination (mine is a tab)

import usb.core
import usb.util

char_dict = {216172799293652992: 'a', 216172803588620288: 'b', 216172807883587584: 'c', 216172812178554880: 'd', 216172816473522176: 'e', 216172820768489472: 'f', 216172825063456768: 'g', 216172829358424064: 'h', 216172833653391360: 'i', 216172837948358656: 'j', 216172842243325952: 'k', 216172846538293248: 'l', 216172850833260544: 'm', 216172855128227840: 'n', 216172859423195136: 'o', 216172863718162432: 'p', 216172868013129728: 'q', 216172872308097024: 'r', 216172876603064320: 's', 216172880898031616: 't', 216172885192998912: 'u', 216172889487966208: 'v', 216172893782933504: 'w', 216172898077900800: 'x', 216172902372868096: 'y', 216172906667835392: 'z', 216735749247074304: 'A', 216735753542041600: 'B', 216735757837008896: 'C', 216735762131976192: 'D', 216735766426943488: 'E', 216735770721910784: 'F', 216735775016878080: 'G', 216735779311845376: 'H', 216735783606812672: 'I', 216735787901779968: 'J', 216735792196747264: 'K', 216735796491714560: 'L', 216735800786681856: 'M', 216735805081649152: 'N', 216735809376616448: 'O', 216735813671583744: 'P', 216735817966551040: 'Q', 216735822261518336: 'R', 216735826556485632: 'S', 216735830851452928: 'T', 216735835146420224: 'U', 216735839441387520: 'V', 216735843736354816: 'W', 216735848031322112: 'X', 216735852326289408: 'Y', 216735856621256704: 'Z', 216172949617508352: '0', 216172910962802688: '1', 216172915257769984: '2', 216172919552737280: '3', 216172923847704576: '4', 216172928142671872: '5', 216172932437639168: '6', 216172936732606464: '7', 216172941027573760: '8', 216172945322541056: '9', 216173009747050496: '`', 216735860916224000: '!', 216735955405504512: '"', 216735873801125888: '$', 216735878096093184: '%', 216735882391060480: '^', 216735886686027776: '&', 216735890980995072: '*', 216735895275962368: '(', 216735899570929664: ')', 216172975387312128: '-', 216735925340733440: '_', 216172979682279424: '=', 216735929635700736: '+', 216172983977246720: '[', 216172988272214016: ']', 216735933930668032: '{', 216735938225635328: '}', 216173001157115904: ';', 216735951110537216: ':', 216173005452083200: "'", 216735865211191296: '@', 216735869506158592: '#', 216173014042017792: ',', 216173018336985088: '.', 216173022631952384: '/', 216172992567181312: '\\', 216735963995439104: '<', 216735968290406400: '>', 216735972585373696: '?', 216172971092344832: ' ', 216172953912475648: "\t"}


#CHANGE TO YOUR SCANNER'S ID CODE (FROM lsusb)
# decimal vendor and product values
#dev = usb.core.find(idVendor="0581", idProduct="0106")
# or, uncomment the next line to search instead by the hexidecimal equivalent
dev = usb.core.find(idVendor=0x581, idProduct=0x106)

# first endpoint
interface = 0
endpoint = dev[0][(0,0)][0]
# if the OS kernel already claimed the device, which is most likely true
# thanks to http://stackoverflow.com/questions/8218683/pyusb-cannot-set-configuration
if dev.is_kernel_driver_active(interface) is True:
  # tell the kernel to detach
  dev.detach_kernel_driver(interface)
  # claim the device
  usb.util.claim_interface(dev, interface)
return_string=''

while True :
    try:
        data = dev.read(endpoint.bEndpointAddress,endpoint.wMaxPacketSize)
        hexstr = ''
        for b in data:
          hexstr += ('{:02x}'.format(b))
        intstr = int("0x{}".format(hexstr), 0)
        if intstr in char_dict:
          if char_dict[intstr] == "\t":#end of line (tab character for me) - alter to your brand's EOL character
            print (return_string)
            return_string = ''
          else:
            return_string += (char_dict[intstr])
    except usb.core.USBError as e:
        data = None
        if e.args == ('Operation timed out',):
            continue
# release the device
usb.util.release_interface(dev, interface)
# reattach the device to the OS kernel
dev.attach_kernel_driver(interface)
