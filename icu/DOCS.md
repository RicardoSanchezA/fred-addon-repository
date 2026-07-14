# ICU Engine

ICU Engine is the native inference backend for the ICU Home Assistant
integration. Install and start the add-on, then add ICU from **Settings >
Devices & services**. Supervisor discovery supplies the private endpoint and
credential automatically.

The add-on stores its instance identity, API credential, accepted
configuration, and durable command state under `/data`, which Home Assistant
includes in add-on backups.

## Options

### `observer_mode`

When enabled, ICU computes and logs every lighting decision exactly as it
normally would, but never calls a real Home Assistant service -- nothing
about your home ever changes. Use this to compare ICU's decisions against
whatever is currently controlling your lights before trusting it with real
control. This is not time-bounded: leave it on for as long as you want to
observe, and turn it off (and restart the add-on) once you're ready for ICU
to actually control lights.
