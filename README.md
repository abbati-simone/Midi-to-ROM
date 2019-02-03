# Midi-to-ROM
Extract Verilog/Quartus-MIF ROM from Midi files

[**Midi2ROM.py**](https://github.com/abbati-simone/Midi-to-ROM/blob/master/src/Midi2ROM.py) create a Verilog or MIF file (Memory initialization file) from a Midi file.

The script needs Python3 *mido* and *getopt* packages.

This script have been used on this project: https://github.com/abbati-simone/Yoshis-Nightmare-Altera
and this one: https://github.com/abbati-simone/Midi-Altera

[**run_conversion.sh**](https://github.com/abbati-simone/Midi-to-ROM/blob/master/src/run_conversion.sh) is a shell script with many examples for included midi files.

Midi2ROM.py example
-------------------
Create a MIF file and parameters from Yoshi_Title.mid using 120 Beats per minute, numbering ROMS sequentially from 0, extracting only track number 2 (the third one) and using "Yoshi" as name prefix.
```bash
$./src/Midi2ROM.py -f quartus -i doc/midi/Yoshi_Title.mid -o doc/mif/ -p doc/parameters/ -b 120 -n 1 -r Yoshi -t2
```
Output *ROM_notes_Yoshi_0.mif* (partial preview):
```
WIDTH = 16;
DEPTH = 146;
ADDRESS_RADIX = HEX;
DATA_RADIX = BIN;
CONTENT BEGIN
0: 1010011111000000;
1: 0010011001011000;
2: 1010000000000000;
3: 0010000001111000;
4: 1010010000000000;
5: 0010010001111000;
6: 1010100000000000;
7: 0010100001111000;
8: 1011000000000000;
9: 0011000101101000;
a: 1010010000000000;
b: 0010010001111000;
c: 1001010000000000;
d: 0001010001111000;
e: 1001000000000000;
f: 0001000001111000;
10: 1001010000000000;
11: 0001010001111000;
12: 1010010000000000;
13: 0010010001111000;
...............
```

Create a Verilog (entire module, not only the *case* portion) file and parameters from Yoshi_Title.mid using 120 Beats per minute, numbering ROMS sequentially from 0, extracting only track number 2 (the third one) and using "Yoshi" as name prefix.
```bash
$./src/Midi2ROM.py -f verilog -i dic/midi/Yoshi_Title.mid -o doc/verilog/ -p doc/parameters/ -b 120 -n 1 -r Yoshi -t2
```
Output *ROM_notes_Yoshi_0.v* (partial preview):
```
module notes_sequence_Yoshi_0(
	input clk,
	input [7:0] address,
	output reg [4:0] note,
	output reg note_on,
	output reg [9:0] delay
);

reg [15:0] note_;

always @(posedge clk) begin
case(address)
0:note_<=16'b1010011111000000;
1:note_<=16'b0010011001011000;
2:note_<=16'b1010000000000000;
3:note_<=16'b0010000001111000;
4:note_<=16'b1010010000000000;
5:note_<=16'b0010010001111000;
6:note_<=16'b1010100000000000;
7:note_<=16'b0010100001111000;
8:note_<=16'b1011000000000000;
9:note_<=16'b0011000101101000;
10:note_<=16'b1010010000000000;
11:note_<=16'b0010010001111000;
12:note_<=16'b1001010000000000;
13:note_<=16'b0001010001111000;
.............
default: note_ <= 16'b0;
endcase

note_on <= note_[15:15];
note <= note_[14:10];
delay <= note_[9:0];
end
endmodule
```

Output *Parameters_Yoshi_general.vh*:
```
parameter ROMS_number=1;
parameter note_max_bits=5;
parameter address_max_bits=8;
parameter delay_max_bits=10;
parameter ticks_per_beat=240;
parameter BPM=120;
parameter BPS=2.0;
parameter ticks_hz=480;
parameter delay_clocks_per_tick=1000;
parameter delay_clock_hz=480000;
parameter delay_reg_bits=29;
```

Output *Parameters_ROM_Yoshi_0.vh*:
```
parameter ROM_0_messages_len=146;
parameter ROM_0_note_min=67;
parameter ROM_0_note_max=84;
parameter ROM_0_delay_min=0;
parameter ROM_0_delay_max=960;
parameter ROM_0_messages_address_bits=8;
parameter ROM_0_note_nbit=5;
parameter ROM_0_delay_nbit=10;
```

**To understand how use these informations check projects linked above.**
Just notice that ROM_0_note_nbit + ROM_0_delay_nbit is 15 and not 16 as showed in the ROM word size, because we need one more bit to store note_on/note_off information.

Output recorded from my development board buzzer using ROMs extracted from *Mario-Sheet-Music-Overworld-Main-Theme.mid* file (partial):
![Output board buzzer record](https://github.com/abbati-simone/Midi-to-ROM/blob/master/doc/Output_board_buzzer_record.mp3 "Output board buzzer record")

Known problems, limitations and possible improvements
-----------------------------------------------------
I didn't notice any problem using this script.
Only midi files that are monotonic are supported (only one note at a time within a track is played). The 'tempo' inside midi file is ignored and only one fixed tempo (via BMP) can be specified as command line parameter. Every instrument used inside a track are extracted to the same ROM.
I tried to reduce the ROM size "width" but this can be likely further improved. For example the note to play is stored subtracting the note offset (the lowest note, notice the *ROM_0_delay_min* parameter).

