import alsaaudio
from audioop import rms
import json
import requests

access_key = ''  # Your wit.ai key should go here.
THRESHOLD = 2000
CHUNK_SIZE = 128


# Returns if the RMS of block is less than the threshold
def is_silent(block):
    try:
        rms_value = rms(block, 2)
        return rms_value, rms_value <= THRESHOLD
    except:
        return -1, True
        # Sometimes during loud noise you don't get the full
        # number of blocks, which causes rms to bug out.


# Returns as many (up to returnNum) blocks as it can.
def returnUpTo(iterator, values, returnNum):
    if iterator+returnNum < len(values):
        return (iterator + returnNum,
                "".join(values[iterator:iterator + returnNum]))
    else:
        temp = len(values) - iterator
        return (iterator + temp + 1, "".join(values[iterator:iterator + temp]))


# Yields chunks for requests to send out.
def gen(inp):
    num_silent = 0
    snd_started = False
    start_pack = 0
    counter = 0
    print "Microphone on!"
    i = 0
    data = []

    while 1:
        l, snd_data = inp.read()
        data.append(snd_data)

        rms, silent = is_silent(snd_data)

        if silent and snd_started:
            num_silent += 1

        elif not silent and not snd_started:
            i = len(data) - CHUNK_SIZE*2  # Set the counter back a few seconds
            if i < 0:                     # so we can hear the start of speech.
                i = 0
            snd_started = True
            print "TRIGGER at " + str(rms) + " rms."

        elif not silent and snd_started and not i >= len(data):
            i, temp = returnUpTo(i, data, 16)
            yield temp
            num_silent = 0

        if snd_started and num_silent > 100:
            print "Stop Trigger"
            break

        if counter > 1000:  # Slightly less than 10 seconds.
            print "Timeout, Stop Trigger"
            break

        if snd_started:
            counter = counter + 1

    # Yield the rest of the data.
    print "Pre-streamed " + str(i) + " of " + str(len(data)) + "."
    while (i < len(data)):
        i, temp = returnUpTo(i, data, 128)
        yield temp
    print "Swapping to thinking."


if __name__ == '__main__':
    inp = alsaaudio.PCM(alsaaudio.PCM_CAPTURE, mode=alsaaudio.PCM_NORMAL)
    inp.setchannels(1)
    inp.setrate(16000)
    inp.setformat(alsaaudio.PCM_FORMAT_S16_LE)
    inp.setperiodsize(CHUNK_SIZE)

    headers = {'Authorization': 'Bearer ' + access_key,
               'Content-Type': 'audio/raw; encoding=signed-integer; bits=16;' +
               ' rate=16000; endian=little', 'Transfer-Encoding': 'chunked'}
    url = 'https://api.wit.ai/speech'

    foo = requests.post(url, headers=headers, data=gen(inp))
    print foo.text
    print "Done."
