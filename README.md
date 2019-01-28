# linux-bluetooth

Workaround for a race condition observed with Android phones which makes fail
some connections (30% with my device).

## Details

After link key reply it is expected that HCI_EV_ENCRYPT_CHANGE is received before any ACL connection request or it will be dropped by bluez L2CAP layer.

The events are received from usb interrupt transfer and ACL packet from usb bulk transfer. I guess the bluetooth controller does not handle correctly write ordering between those two transfer types.

The connection/disconnection are tracked so the workaround can apply with a per device logic. It will start queueing ACL packets when link key are exchanged until the HCI_EV_ENCRYPT_CHANGE event or a disconnection.

## POSSIBLE LIMITATIONS

- Connection requests made between LINK_KEY_REQ and ENCRYPT_CHANGE events will be forwarded to the L2CAP layer and will be seen as legitimate.
However, bluetooth specification says that ACL transfer is paused until encryption is fully setup. I guess this is the responsability of the controller to filter any rogue connection request until ENCRYPT_CHANGE event.

- This workaround is not enough to fix a similar issue with pairing.
