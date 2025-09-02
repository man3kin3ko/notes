## GUIDs v1

GUIDs are defined in [RFC4122](https://datatracker.ietf.org/doc/html/rfc4122). They have a specific structure, which also points to its version.
The first version was calculated based upon those values:
- The current time
- A “_clock sequence_” that is randomly generated on boot up, but which remains constant between GUIDs
- A “_node ID_“, which is generated based on the system’s MAC address
If the 13th character (the first one after the second hyphen) is equal to one, that means that it was calculated based on those values, which can be easily guessed. With a 2 second sliding window to account for drift there will be 4,000 possible GUIDs to try.

If application uses newer versions of GUIDs, also check the backward compatibility. 
### See also
- https://danaepp.com/attacking-predictable-guids-when-hacking-apis

## Developer codes

### OTP

```
1111
1234
0000
123456
000000
111111
```