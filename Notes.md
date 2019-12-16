# Useful notes

*from Philippe Gauthier about the offset of the timestamps*
There's clearly a gap there, but there is at least an interesting post in the ANT forum. ANT is roughly the organization that manages the ANT+ standard and FIT formats, led by Garmin. An employee acknowledges the lack of documentation and explains how to calculate the timestamp by providing a snippet of code.

https://www.thisisant.com/forum/viewthread/6374/

This is Java code, but in Python the syntax would be exactly the same except for the semicolon at the end of the line.

I hope it's not too difficult to get the first timestamp in UNIX timestamp format from the FIT file (possibly with `raw_value`?) Those timestamps are 32-bit integers. From there, it's possible to apply the transformations. It probably would look at bit like the following. For each record, print the UNIX timestamp:

for record in fitfile.get_messages():
    if record.type != 'data':
        continue for field_data in record:
    if field_data.name == 'timestamp':
        timestamp = field_data.raw_value
    elif field_data.name == 'timestamp_16':
        timestamp_16 = field_data.raw_value
        timestamp += (timestamp_16 - (timestamp & 0xffff)) & 0xffff
    print(timestamp)
    # probably that works too: print(datetime.datetime.utcfromtimestamp(631065600 + timestamp))



The magic here is the way it handles rollovers, that is, when the timestamp_16 counter exceeds 2**16 and starts at zero again. The first part of the operation is to subtract the lower 16 bits of the previous timestamp from the new timestamp_16 value. There are two possible outcomes. If there is no rollover, the result is the difference in seconds between the previous timestamp and the new one, so it's added to the previous timestamp and we get the new value. If there is a rollover happening for this record, the result of the subtraction is negative because timestamp_16 got smaller. The "& 0xffff" at the end does a modulo operation (mod 2**16) that retrieves the correct difference. It can then be added to the previous timestamp the same way.

I'll try to find a file that contains such timestamps. Maybe my watch is able to generate such files, I simply haven't tried. Sometimes there's a sample in the Garmin FIT SDK.

