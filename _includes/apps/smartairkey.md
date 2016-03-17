# [SmartAirkey](https://itunes.apple.com/ru/app/smartairkey/id1032832416?l=ru)

## iOS application that allows you to open electronic locks by your smartphone.

## Tricky parts

* Database
	* Secure database with delayed setup.
* Network
	* Message-based network adapter with offline mode and optimistic locks.
* Bluetooth
	* Session-based bluetooth facade with state machine inside.
	* Tricky monitoring rssi policy.
	* RSSI-based algorithms.
* Transport
	* Secure features like jwt/ssl-like handshakes.
* UI
	* Event-based updates with event machine.
	* True iOS-styled UI elements. ( tabbars/navbars/tableviews and their customizations, Lock screen controller)

* **Technologies**
	* encrypted-sqlite, jwt, core-bluetooth, gzip, core-data, state machine, event bus, fluent, objective-c/swift

* **Libraries**
	* LITEventMachine, TransitionKit, AFNetworking, EncryptedCoreData, LITLightweightStore




