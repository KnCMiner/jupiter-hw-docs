KnC SPI Protocol specification
==============================

The protocol is a full duplex SPI protocol. Response status header is coming
immediately at first clocked bit.

Request and response formats are aligned in such way that there is three
response packets transmitted per request (384 bits / 96 bits = 3)

Requests
--------

Requests are sent as an array of up to N frames of 384 bits, each having the
same format.

Frame format, MSB first

      4    12       15                      352
    +---+-+-------+---------+---------------------------------------------+
    |CMD|0| QUEUE |   ID    |        DATA                                 |
    +---+-+-------+---------+---------------------------------------------+

CMD 	Command

* NOP(0)		No operation
* VERSION(1)		Returns version information
* SUBMIT_WORK(2)	
* FLUSH(3)

0	Reserved bit. Always 0.

QUEUE	Queue ID. Not yet implemented. Should always be sent as 0.

ID	Request ID

DATA	Work state

QUEUE & ID together builds the identifier of a specific work and needs
to be requrned in status responses.

Responses
---------

Responses always starts with 64 bit header, followed by 96 bit responses

Header contents

	  31               1          16         16               32
    +----------------+----------+----------+-----------------+------------+
    | reserved flags | Overflow | reserved | work slots free | reserved   |
    +----------------+----------+----------+-----------------+------------+


- Overflow error indication bit. Some responses have been lost if this bit
  is set.
- Free queue space. If more SUBMIT_WORK requests is sent in the same
  transaction then at most this number of work items will have been
  consumed, the rest should be resubmitted at a later time.

Response contents

- Response type, NOP/HIT/DONE (2 bits)
- ASIC #, 0-5 (3 bits)
- Queue
- Id
- Nonce
- Core #

Response frame format, MSB first

      2    3     12       15           32
    +----+----+-------+---------+-------------------+-------------------+
    |TYPE|ASIC| QUEUE |   ID    |      NONCE        |       CORE        |
    +----+----+-------+---------+-------------------+-------------------+


TYPE Response Type

* NOP (0), empty slot
* HIT (1), NONCE HIT
* DONE (3), WORK DONE/COMPLETED

ASIC Asic # that handled the work

QUEUE Queue ID from SUBMIT_WORK

ID Work ID from SUBMIT_WORK

NONCE Nonce value (only HIT responses, #0 on DONE responses)

CORE Core # that handled the work within the ASIC

Note on hashing progress indication: There is no specific hashing progress
indication, each DONE reports indicate 2^32 hashing operations have been
performed. If work is flushed then there will be no report indications of
the flushed work.
