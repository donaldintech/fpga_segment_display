# Implementing a 4 Digit Display on Cyclone II Altera FPGA Development Board.
## Background Info ##
The DE1 Board has four 7-segment displays. These displays are arranged into a group of four, with the intent of displaying numbers of various sizes. The seven segments are connected to pins on the Cyclone II FPGA (refer to DE1 manual for the specific pins) ; where each segment in a display is identified by an index from 0 to 6.
Applying a low logic level to a segment causes it to light up, and applying a high logic level turns it off. 

Generally, there are two types of LED 7-segment display called: Common Cathode (CC) and Common Anode (CA).
The difference between the two displays, as their name suggests, is that the common cathode has all the cathodes of the 7-segments connected directly together and the common anode has all the anodes of the 7-segments connected together and is illuminated as follows.
![image](https://github.com/user-attachments/assets/03e5e1a4-0d4f-45b6-b814-4802f85cbf85)
![image](https://github.com/user-attachments/assets/d1e808a1-4b13-43ad-a4d5-c4aeada89fb4)

 

Fig.1: Common Cathode & Common Anode(credits: electronics-tutorials blog)


The EP2C20F484C7N board that we’re using for this tutorial has a CA configuration; Anode  connected to 3.3 V while all the LED cathode segments are grounded. We’ll therefore have this in mind as we implement this project.

Our project’s goal is to implement a counter utilizing these 4 seven segment displays


## Requirements & Installation ##

1.	Quartus Web II software (This project uses ver. 13.0.1)
2.	Cyclone II FPGA Development kit (EP2C20F484C7N)

•	Install the software from Intel’s Downloads section Intel as shown 

![image](https://github.com/user-attachments/assets/4f81003a-d090-4d57-a396-9aed63d58f8a)

 
•	EXTRACT the files into the same temporary directory.<br/>
•	RUN the setup.bat file.<br/>
•	You can then run the exe. file .

 ![image](https://github.com/user-attachments/assets/7a5f3964-ffc9-4ad2-8b70-fc28edd19eaa)


•	From the file drop down menu, select ‘NEW PROJECT WIZARD’<br/>
•	On the next window, select where you want to have your project, preferably a new folder

 ![image](https://github.com/user-attachments/assets/5845fa5b-1ee0-4d70-b1dc-d22a745cf523)


•	Add files if you have any then proceed to select the device settings

 ![image](https://github.com/user-attachments/assets/81131cb0-c658-4eed-93e9-ef2151e46b67)


•	Your project is then set , so click FINISH.<br/>
•	Now that we’re in the project, click again on the files dropdown menu and select VHDL file, then SAVE it with a suitable name.

 ![image](https://github.com/user-attachments/assets/bd1600f0-f970-4449-97db-c1ba158903fb)


You are now set to begin coding for the project.


## Programming in VHDL ##

Breaking down our program into sections provide an easier way to understand what each section does and represent.

**•	Libraries**

Define the standard libraries which facilitate design reuse and standard data types for model exchange, reuse, and synthesis. 

![image](https://github.com/user-attachments/assets/fb4c4dcd-5d9a-497f-997a-111cc68f5e1f)

 
**•	Entity**

Define an entity which contains all the input & output ports to be used in the program

![image](https://github.com/user-attachments/assets/a5299f7b-ab71-444c-a4ff-b2c27666e469)

 
**•	Architecture**

The main part of our program, contains two sections: 

**1.Signal declaration** 

-This is where you declare all the signals and variables to be used in the processes that follow

 ![image](https://github.com/user-attachments/assets/40ed0089-cd98-438e-b452-b18fe918138a)


 **2.Process definitions** 

•	Synchronize button signal

-Our first process is to synchronize the button input signal(btn) with the fpga clock signal(clk) since button presses are asynchronous events, and directly using them in synchronous logic can lead to metastability and unpredictable behavior.
![image](https://github.com/user-attachments/assets/134698fb-4178-4428-a0e0-cb63333fe87c)

 
-The reset logic ensures the synchronized button signal starts from a safe initial condition, then during a rising edge of the clock signal, the current state of the least significant bit (btn_sync(0)) is concatenated with the current button input (btn) to obtain a new synchronized button signal.
-This shifts the previous button state to the left and inserts the current button state as the new least significant bit; thus providing a stable signal to be used in subsequent processes.

**•	Debounce button press**

-This process ensures that the system detects a stable and clean button press without any false triggers (bounced )

 ![image](https://github.com/user-attachments/assets/668c4eae-fb3a-45f7-8175-afc553a68758)


-The reset sets the debounced button to an off state(1) while a rising edge of the clock ensures that when the new value of synchronized button signal is asserted low (0), it is matched to the debounced signal, representing a button press.

**•	Detect button press**

-The process ensures the system detects a stable and valid button press event, avoiding multiple or false detections.

![image](https://github.com/user-attachments/assets/c2c6af43-29b4-45b6-ab22-ec6000c5fe0e)

 
-A low debounced signal and a stable synchronized signal provide conditions for a valid detection of a button press (0).

**•	Synchronize and Debounce reset signal**

-After synchronizing & debouncing the button input, we can now do the same for the reset button.

  ![image](https://github.com/user-attachments/assets/49ed6041-2bb7-4b68-a3da-3675b4cf8a9e)


**•	Counter Logic**

-Now to our final 2 main processes, we need to understand how our counter will be implemented  and later displayed on the board.

![image](https://github.com/user-attachments/assets/f5ad1707-3177-4b2c-ade0-daf535f93c69)

  
-For every rising edge of the clock cycle, if a button press is detected, count value is incremented by 1 as long as it is not more than 9999.  
-The digits are then updated using the ‘mod’ operator which returns a remainder of a division operation i.e: if 4356 is the count value, count mod 10 will be 6, (count /10 ) mod 10 will be 5 etc.

**•	Display Logic with Multiplexing**

-The purpose of this process is to sequentially activate each of the four 7-segment displays (digits) to show a multi-digit number. This is done by rapidly switching between each digit, giving the illusion that all four digits are lit simultaneously.

 ![image](https://github.com/user-attachments/assets/7938801a-1b36-4fa5-9d7f-54cf8db1bb6b)


-When the reset is asserted, all signals & segments are turned off but during the rising edge of the cock cycle; the period_count signal (which represents number of clock cycles and is computed from 50 MHz clock and 0.5 mS period)  increments until 25000, meaning it keeps displaying for that long.<br/>
-Otherwise the period is reset to 0 and thus the display_sel signal determines the segment to display using a modulo operator; whose result is utilized in the case statement.<br/><br/>
-FIND THE WHOLE CODE BELOW: 

```
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity four_digit is
port (
    clk : in std_logic;   -- Clock signal
    rst : in std_logic;   -- Reset signal
    btn : in std_logic;   -- Pushbutton input

    seg1 : out std_logic_vector(6 downto 0);  -- 7-segment display 1 (units place)
    seg2 : out std_logic_vector(6 downto 0);  -- 7-segment display 2 (tens place)
    seg3 : out std_logic_vector(6 downto 0);  -- 7-segment display 3 (hundreds place)
    seg4 : out std_logic_vector(6 downto 0);  -- 7-segment display 4 (thousands place)
    dp   : out std_logic   -- Decimal point (not used here, set to '1')
);
end entity;

architecture rtl of four_digit is

-- Define an array type for the 7-segment display patterns
type seg_array_type is array (0 to 9) of std_logic_vector(6 downto 0);

-- Define the 7-segment display patterns for digits 0-9
constant seg_patterns : seg_array_type := (
    "1000000", -- 0
    "1111001", -- 1
    "0100100", -- 2
    "0110000", -- 3
    "0011001", -- 4
    "0010010", -- 5
    "0000010", -- 6
    "1111000", -- 7
    "0000000", -- 8
    "0010000"  -- 9
);

-- Define the pattern for all segments off
constant all_off : std_logic_vector(6 downto 0) := "1111111";

-- Signal Declarations
signal count        : integer := 0;
signal btn_sync     : std_logic_vector(1 downto 0);
signal btn_debounce : std_logic;
signal btn_pressed  : std_logic;
signal rst_sync     : std_logic_vector(1 downto 0);
signal rst_debounce : std_logic;

-- Signals for multiplexing
signal display_sel   : integer := 0; -- For multiplexing display
signal period_count  : integer := 0; -- For timing multiplexing

-- Signals for 4-digit counter
signal unit         : integer := 0;
signal tens          : integer := 0;
signal hundreds      : integer := 0;
signal thousands     : integer := 0;

begin

    -- Initialize decimal point to '1' (not used)
    dp <= '1';

    -- Synchronize button signal
    sync_btn: process(clk, rst)
    begin
        if rst = '0' then
            btn_sync <= "00";
        elsif rising_edge(clk) then
            btn_sync <= btn_sync(0) & btn;
        end if;
    end process;

    -- Debounce button press
    debounce_btn: process(clk, rst)
    begin
        if rst = '0' then
            btn_debounce <= '1';  -- Set to '1' because button is not pressed
        elsif rising_edge(clk) then
            if btn_sync(1) = '0' then
                btn_debounce <= '0';  -- Button pressed
            else
                btn_debounce <= '1';  -- Button not pressed
            end if;
        end if;
    end process;

    -- Detect button press
    detect_btn_press: process(clk, rst)
    begin
        if rst = '0' then
            btn_pressed <= '1';  -- No button press during reset
        elsif rising_edge(clk) then
            if btn_debounce = '0' and btn_sync(1) = '1' then
                btn_pressed <= '0';  -- Button press detected
            else
                btn_pressed <= '1';  -- No button press
            end if;
        end if;
    end process;

    -- Synchronize and debounce reset signal
    sync_rst: process(clk, rst)
    begin
        if rst = '0' then
            rst_sync <= "00";
        elsif rising_edge(clk) then
            rst_sync <= rst_sync(0) & rst;
        end if;
    end process;

    debounce_rst: process(clk, rst)
    begin
        if rst = '0' then
            rst_debounce <= '0';  -- Active low reset
        elsif rising_edge(clk) then
            if rst_sync(1) = '0' then
                rst_debounce <= '0';  -- Reset is active
            else
                rst_debounce <= '1';  -- Reset is not active
            end if;
        end if;
    end process;

    -- 4-digit Counter Logic
    counter_logic: process(clk, rst)
    begin
        if rst_debounce = '0' then
            count <= 0;
            unit <= 0;
            tens <= 0;
            hundreds <= 0;
            thousands <= 0;
        elsif rising_edge(clk) then
            if btn_pressed = '0' then
                if count < 9999 then
                    count <= count + 1;
                else
                    count <= 0;
                end if;

                -- Update individual digits
                unit <= count mod 10;
                tens <= (count / 10) mod 10;
                hundreds <= (count / 100) mod 10;
                thousands <= (count / 1000) mod 10;
            end if;
        end if;
    end process;

    -- Display Logic with Multiplexing
    multiplexing_display: process(clk, rst)
    begin
        if rst = '0' then
            period_count <= 0;
            display_sel <= 0;
            seg1 <= all_off;  -- Default to all segments off
            seg2 <= all_off;  -- Default to all segments off
            seg3 <= all_off;  -- Default to all segments off
            seg4 <= all_off;  -- Default to all segments off
        elsif rising_edge(clk) then
            if period_count < 25000 then
                period_count <= period_count + 1;
            else
                period_count <= 0;
                display_sel <= (display_sel + 1) mod 4;
            end if;

            case display_sel is
                when 0 =>
                    seg1 <= seg_patterns(unit);      -- Units place
                    seg2 <= all_off;                  -- Turn off tens place
                    seg3 <= all_off;                  -- Turn off hundreds place
                    seg4 <= all_off;                  -- Turn off thousands place
                when 1 =>
                    seg1 <= all_off;                  -- Turn off units place
                    seg2 <= seg_patterns(tens);       -- Tens place
                    seg3 <= all_off;                  -- Turn off hundreds place
                    seg4 <= all_off;                  -- Turn off thousands place
                when 2 =>
                    seg1 <= all_off;                  -- Turn off units place
                    seg2 <= all_off;                  -- Turn off tens place
                    seg3 <= seg_patterns(hundreds);   -- Hundreds place
                    seg4 <= all_off;                  -- Turn off thousands place
                when 3 =>
                    seg1 <= all_off;                  -- Turn off units place
                    seg2 <= all_off;                  -- Turn off tens place
                    seg3 <= all_off;                  -- Turn off hundreds place
                    seg4 <= seg_patterns(thousands);  -- Thousands place
                when others =>
                    seg1 <= all_off;                  -- Turn off all places
                    seg2 <= all_off;                  -- Turn off all places
                    seg3 <= all_off;                  -- Turn off all places
                    seg4 <= all_off;                  -- Turn off all places
            end case;
        end if;
    end process;

end architecture;

``` 





## Preparing the Code ##

-Click on the SIMULATE icon to ensure no errors exist in the code and monitor the following section of the software window, which shows the progress.

![image](https://github.com/user-attachments/assets/45c9c377-3b10-4117-9238-4e086b12e27b)


-If everything is green, click on the PIN PLANNER icon to specify FPGA pins. <br/>
A new window appears where you can also specify the Voltage levels alongside the location of the pins; all with the help of the User Manual or Datasheet.

![image](https://github.com/user-attachments/assets/28d17c3a-8ece-471a-a189-6dd31f523092)


-SIMULATE one more time to ensure the pins are allocated.<br/>
-The final section is to program the FPGA ; Click on the PROGRAMMER icon which opens a new window

![image](https://github.com/user-attachments/assets/a84de246-2dbc-4779-82cd-3ceaf1effa62)


-You might need to setup your hardware if it does not show up. Some FPGA boards use an external USB blaster but ours is embedded-so if you have any problem here you need to update drivers in the Device Manager automatically or manually by navigating to the location of the downloaded Altera package.

 ![image](https://github.com/user-attachments/assets/ae85f031-d358-4e6d-a6f3-11023d5ac519)


-If everything is okay, click on ADD FILE , then click on the .sof file which has a volatile memory, therefore disconnecting the board loses the program; otherwise go for the .pof file.

-Click on START  and you should see a green progress bar

 ![image](https://github.com/user-attachments/assets/12e4cd6e-1cf4-430b-9b6a-bb800f9ba2e5)


-The board should now behave as programmed.

