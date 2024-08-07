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





The Overall Code is attached below:

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
            btn_pressed <= '0';  -- No button press during reset
        elsif rising_edge(clk) then
            if btn_debounce = '0' and btn_sync(1) = '1' then
                btn_pressed <= '1';  -- Button press detected
            else
                btn_pressed <= '0';  -- No button press
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
            if btn_pressed = '1' then
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
