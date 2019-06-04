# Lyse Cell

The objective of this protocol is to lyse cell by a sonicator and then spin the sample to collet supernatant for applying to a packed column.

- Workflow:
  - Column Packing > *Lyse cell* > His-tagged Purification
- Steps:
  - [01] Grab cell pellet from -80°C freezer and thaw it on ice.
  - [02] Prepare lysis buffer.
  - [03] Suspend cell pellet in lysis buffer and incubate for 20 minutes on ice.
  - [04] Lyse cell by a sonicator.
  - [05] Spin down cell with a high speed centrifuge.
  - [06] Keep supernatant and discard the cell pellet.
### Inputs


- **Cell Pellet** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Cell Pellet of protein")'>Cell Pellet of protein</a>



### Outputs


- **Cell Lysate** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Cell Lysate of Protein Expression")'>Cell Lysate of Protein Expression</a>

### Precondition <a href='#' id='precondition'>[hide]</a>
```ruby
def precondition(_op)
  true
end
```

### Protocol Code <a href='#' id='protocol'>[hide]</a>
```ruby
# Author: Pei Wu , Sep 2018

needs "Standard Libs/Feedback"

class Protocol
  include Feedback
  def main
    operations.retrieve.make
    
    op_in_cell_pellet = []
    op_out_cell_lysate = []
    op_count = 0
    operations.running.each do |op|
        op_count = op_count + 1
        op_in_cell_pellet << op.input("Cell Pellet").item.id
        op_out_cell_lysate << op.output("Cell Lysate").item.id
    end
    
     # Take the cell paste from -80°C and let cells thaw on ice.
    take_cell_pellet(op_in_cell_pellet)
    
    # Take and add 50mL lysis buffer into a 50mL Falcon tube and add one tablet of protease inhibitor.
    take_lysis_buff(op_count,op_in_cell_pellet)
        
    # Shake the Falcon tube and mix the buffer well until the protease inhibitor tablet fully dissolved in the lysis buffer.
    distribute_lysis_buff(op_count,op_in_cell_pellet)
        
    # Lyse the cell by a sonicator in Prof. Seelig’s Lab.
    lyse_cell_by_sonicator(op_count,op_in_cell_pellet,op_out_cell_lysate)
        
    # Turn on the sonicator and set the time and temperature condition, 95% power output, 10secs on, 30secs off, repeat 30 rounds.
    set_up_sonicator(op_count)
    
    add_binding_buffer(op_count)
    
    weigh_and_balance_tubes(op_count)
         
    # Spin by the tabletop high-speed centrifuge under 24,000g for 30mins at 4°C.
    spin_hs_centrifuge(op_count)
        
    # Transfer the supernatant to new tubes and discard the cell pellet.
    transfer_supernatant_to_new_tubes_label(op_count,op_in_cell_pellet,op_out_cell_lysate)
    
    operations.running.each do |op|
      op.output("Cell Lysate").child_item.move "4°C or on ice for the (next) protocol"
    end
    
    operations.store(io: "output", interactive: true)
    get_protocol_feedback

    return {}

  end
  
  def take_cell_pellet(op_in_cell_pellet)
    show do
       title "Thaw cell pellets on ice"
       check "Grab a ice bucket and fill it with ice."
       check "Keep #{op_in_cell_pellet.to_sentence} in ice bath."
    end
  end

  def take_lysis_buff(op_count,op_in_cell_pellet)
    op_table = [["Falcon tube ID","Lysis buffer volume","Protease inhibitor"]]
    for i in 0..(op_count-1)
        row = []
        row << i+1
        row << "20 mL"
        row << "2 tablets"
        op_table << row
    end
    show do
        title "Lysis buffer preparation"
        check "Grab <b>#{op_count}</b> 50 mL Falcon tube and a rack."
        check "Label tube from 1 to #{op_count} on the tube wall and cap."
        check "Grab lysis buffer and protease inhibitor from 4°C refrigerator."
        check "Pour 20 mL of lysis buffer to each tube."
        check "Add 2 tablets of protease inhibitor to each tube by using a tweezer."
        table op_table
        check "Place tubes on platform shaker until the tablets are fully dissovled."
        check "Return lysis buffer and protease inhibitor."
    end
  end

  def distribute_lysis_buff(op_count,op_in_cell_pellet)
    op_table = [["50 mL Falcon tube ID","225 mL Falcon tube ID"]]
    for i in 0..(op_count-1)
        row = []
        row << i+1
        row << {content:op_in_cell_pellet[i], check: true}
        op_table << row
    end
    show do
       title "Suspend pellets in lysis buffer"
       note "Perform the following steps on ice."
       check "Pour lysis buffer into the corresponding 225 mL tube."
       table op_table
       check "Keep the used 50 mL Falcon tubes (ID from 1 to #{op_count}) on a rack for later use."
       check "Grab motorized pipet and <b>#{op_count}</b> 25mL-serological pipette(s)."
       check "Pipet up and down to suspend cell pellets in lysis buffer. In this step, you can scratch off the pellet by the pipette tip, and then suspend it until homogenized. Make sure no cell clumps remain on the pipette tip. No vortexing."
       if op_count <= 2
            check "Incubate #{op_in_cell_pellet.to_sentence} in ice bath for 15 minutes."
            timer initial: { hours: 0, minutes: 15, seconds: 0}
       end
    end
  end
  
  def lyse_cell_by_sonicator(op_count,op_in_cell_pellet,op_out_cell_lysate)
    op_table = [["225 mL Falcon tube ID","50 mL Falcon tube ID"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_cell_pellet[i]
        row << i+1
        op_table << row
    end
    show do
       title "Transfer samples to Falcon tubes"
       check "Pour cell lysate back to the corresponding 50 mL tube."
       table op_table
       check "Place sample tubes in ice bath."
       check "Discard the used 225 mL tubes."
       
    end
    show do
        title "Sonication preparation"
        bullet "Bring the following items to the sonicator."
        check "All samples (ID: 1 to #{op_count}) and the ice bucket."
        check "a 600 mL beaker and marker pen"
        check "Ethanol spray bottle and Kimwipes."
    end
  end
  
  def set_up_sonicator(op_count)
    show do
        title "Set up a sonicator"
        bullet "Turn on the sonicator."
        bullet "Press <b>ENTER</b> to set the processing parameters:"
        note  "95% power delivered to the probe"
        note "10 secs ON, 30 secs OFF, repeat 30 rounds and the total time is 1200 seconds."
        bullet "Spray the sonicator tip with Ethanol and wipe it dry with Kimwipes."
        bullet "Insert the tube (ID:1) in the beaker filled with ice. (as shown in the picture 1) "
        bullet "Use the adjustable knob to immerse the sonicatior tip into the sample (as shown in the picture 2). Leave 2-3 cm space between tip point and the bottom of a tube."
        warning "Sonicator tip should not be touching the bottom or sides of the tube."
        bullet "Press <b>START</b> to activiate sonication."
        bullet "After 30 rounds are done, screw the cap and put it back to the ice bucket. Circle the tube ID on the cap as completed."
        bullet "Spray the sonicator tip with Ethanol and wipe it dry in each time switching to the next tube."
        bullet "Pour off water from the beaker and add fresh ice on the top in each time starting a new cycle of sonication, preventing ice melting too fast. (as shown in the picture 3 & 4)"
        bullet "Perform sonication from tube 1 to #{op_count} until all tubes have been done."
        #warning "A temperature increase in the sample might denature protein. Keep the tube on ice while performing sonication."
        image "Actions/ProteinPurification/sonication_four_steps.jpg"
    end
  end
  
  def add_binding_buffer(op_count)
    show do
        title "Add purification buffer"
        check "Grab purification buffer from 4°C refrigerator."
        check "Add purification buffer up to the 40 mL-mark in each tube (ID from 1 to #{op_count})."
        check "Keep tubes in ice bath."
        check "Return purification buffer to 4°C refrigerator."
    end
  end
    
  def weigh_and_balance_tubes(op_count)
    show do
        title "Scale balancing"
        check "Weigh the tubes. Balance samples in tubes."
        check "If the balance is off, add Purification Buffer into the sample tube until a balance is reached."
        warning "It's very important to balance tubes before using the high-speed centrifuge."
    end
  end
  
  def spin_hs_centrifuge(op_count)
    show do
        title "Spin down cells"
        note "Bring tubes (ID: 1 to #{op_count}) in a ice bucket to a high-speed centrifuge."
        check "Spin at 24,000g for 30 minutes at 4°C."
        timer initial: { hours: 0, minutes: 30, seconds: 0}
        check "Remove tubes from the high speed centrifuge. Keep tubes in ice bath."
        image "Actions/ProteinPurification/high_speed_centrifuge.jpg"
    end
  end
    

  def transfer_supernatant_to_new_tubes_label(op_count,op_in_cell_pellet,op_out_cell_lysate)
    op_table = [["Sample tube ID","column ID"]]
    for i in 0..(op_count-1)
        row = []
        row << i+1
        row << {content:op_out_cell_lysate[i], check: true}
        op_table << row
    end
    
    show do
        title "Transfer samples"
        check "Grab <b>#{op_count}</b> 50 mL Falcon tube(s)."
        check "Label tube(s) with ID: #{op_out_cell_lysate.to_sentence} on the tube wall and cap."
        check "Pour supernatant to new tubes according to the table."
        check "Place tubes in ice bath."
        table op_table
    end
    
    show do
        title "Clean up"
        check "Discard tubes with ID: 1 to #{op_count}."
    end
  end
end

```
