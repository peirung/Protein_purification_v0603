# Harvest Cell

The objective of this protocol is to collect cell pellet by a centrifuge.

- Workflow:
  - IPTG induction > *Harvest Cell* > Column Packing 
- Steps:
  - [01] Retrieve the cell culture from 37°C shaker.
  - [02] Reserve 1 mL of cell culture in a 1.5 mL tube.
  - [03] Pour 200 mL cell culture into a 225 mL Falcon tube.
  - [04] Spin down the cell at 4696g for 15 minutes.
  - [05] Remove the tube from the centrifuge and discard supernatant.  
  - [06] Pour all the remaining cell culture to the tube and spin down the cell at 4696g for another 15 minutes.
  - [07] Pour off supernatant and keep the cell pellet at -80°C freezer.
### Inputs


- **Overexpression** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "TB Overnight (400 mL) of Plasmid")'>TB Overnight (400 mL) of Plasmid</a>



### Outputs


- **After IPTG** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Overexpression of Plasmid")'>Overexpression of Plasmid</a>

- **Cell pellet** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Cell Pellet of protein")'>Cell Pellet of protein</a>

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
        
    # Remove the overexpression culture from the shaker.
    operations.retrieve.make
        
    op_in_overexpression = []
    op_out_cell_pellet = []
    op_out_after = []
    op_count = 0
    operations.running.each do |op|
        op_count = op_count + 1
        op_in_overexpression << op.input("Overexpression").item.id
        op_out_cell_pellet << op.output("Cell pellet").item.id
        op_out_after << op.output("After IPTG").item.id
    end

    # Reserve 1ml cell culture after IPTG induction in Eppendorf with design ID 
    reserve_culture_after_iptg(op_count,op_in_overexpression,op_out_after,op_out_cell_pellet)
    
    take_tubes_and_label(op_count,op_in_overexpression,op_out_cell_pellet)
    
    spin_down_cell(op_count,op_out_after,op_in_overexpression,op_out_cell_pellet)

    # Measure OD600 value cell culture after IPTG induction by nanodrop (Prof. James Carother’s lab).
    od_value_after = measure_od_value op_out_after
        
    # Record the OD value.
    i = 0
    operations.running.each do |op|
        sample_after = op.output("After IPTG").item
        sample_after.associate :od_value, od_value_after[i]
        i = i + 1
    end
    
    spin_down_pellet(op_out_after)
    
    spin_down_remaining_cell(op_count,op_out_after,op_in_overexpression,op_out_cell_pellet)
    
      operations.running.each do |op|
      op.output("Cell pellet").child_item.move "-80°C freezer"
    end
    
    operations.store(io: "output", interactive: true, method: 'boxes')
    
    clean_up
    
    get_protocol_feedback

    return {}
  end


  def measure_od_value(meas_sample)
    od_value = []
    show do
        title "Measure OD600 value of cell culture in MolES lab"
        check "Grab a box to carry the following items:"
        bullet "a box of 1000 µl tip"
        bullet "a P1000 pipettor" 
        bullet "a tip waste container" 
        bullet "#{meas_sample.size+1} plastic cuvettes"
        bullet "1 mL of TB aliquot"
        bullet "a marker pen"
        bullet "sample in a 1.5 mL tube: #{meas_sample.to_sentence}"
    end
    
    show do
        title "Measure OD600 value of cell culture in MolES lab"
        note "Perform the following steps with #{meas_sample.to_sentence}."
        bullet "Open Nanodrop 2000 <b>></b> Home <b>></b> cell culture <b>></b> use cuvette <b>></b> pathlength: 10mm <b>></b> stir speed: off"
        bullet "Blank with 1 mL TB in a cuvette."
        bullet "Discard TB to the liquid waste container."
        bullet "Transfer 1 mL of sample to a new cuvette."
        bullet "Measure OD600 value and record it on the 1.5 mL tube wall."
        bullet "Transfer sample from the cuvette back to its 1.5 mL tube."
        warning "In this step, be careful not to pour samples into the liquid waste container."
        image "Actions/ProteinPurification/cuvette_nanodrop.jpg"
    end
    
  def spin_down_pellet(op_out_after)
    show do
        title "Spin down cell"
        note "Perform the steps with 1.5 mL sample tubes: #{op_out_after.to_sentence}."
        check "Spin down at 8000 g for 3 minutes."
        check "Remove tubes from the centrifuge. Pipette out the supernatant and keep pellet."
        check "Place 1.5 mL sample tubes aside for later processing."
    end
  end
    
    meas_sample.each do |id|
      od = show do
        title "Enter OD value"
        get "number", var: "x", label: "Enter OD600 value of #{id}", default: 0
      end
      od_value << od[:x]
    end
    return od_value
  end

  def reserve_culture_after_iptg(op_count,op_in_overexpression,op_out_after,op_out_cell_pellet)
    op_table = [["Cell culture ID","Volume","1.5 mL tube ID"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_overexpression[i]
        row << "1 mL"
        row << {content:op_out_after[i], check: true}
        op_table << row
    end
    
    show do
        title "Pre-cool a centrifuge"
        check "Set the centrifuge to 4°C."
        image "Actions/ProteinPurification/centrifuge.jpg"
    end
    
    show do
        title "Reserve 1 mL of cell culture before harvesting cell"
        # after IPTG induction
        check "Grab <b>#{op_count}</b> 1.5 mL tube(s)."
        check "Label the tube with ID: #{op_out_after.to_sentence}."
        check "Spray a P1000 pipettor with ethanol and wipe it dry."
        check "Swirl the bottle and transfer 1 mL of cell culture to the corresponding 1.5mL tube."
        note "**Use a new tip for each batch to avoid cross contamination.**"
        check "Place the 1.5 mL sample tubes aside for later task."
        table op_table
    end
  end
  
  def take_tubes_and_label(op_count,op_in_overexpression,op_out_cell_pellet)
    op_table = [["Cell culture ID","Volume","225 mL Falcon tube ID"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_overexpression[i]
        row << "220 mL"
        row << {content:op_out_cell_pellet[i], check: true}
        op_table << row
    end
    
    show do
        title "Prepare tubes"
        check "Grab a ice bucket and fill it with ice."
        check "Grab <b>#{op_count}</b> 225 mL Falcon tube(s) and label with ID: #{op_out_cell_pellet.to_sentence} on the tube wall."
        check "Place tubes in ice bath."
        image "Actions/ProteinPurification/225_falcon_tube.jpg"
    end
    
    show do 
      title "Harvest cell"
      check "Pour approximately 220 mL of cell culture into the corresponding tube and screw the cap."
      table op_table
    end
  end
  
  def spin_down_cell(op_count,op_out_after,op_in_overexpression,op_out_cell_pellet)
      
    run = (op_count/4).floor
    remainder = op_count % 4
    if (remainder) != 0
        run = run + 1
    end
    
    for i in 1..run
        if (i*4 > op_count)
            leng = remainder
        else
            leng = 4
        end
        tube_id = op_out_after[(i-1)*4,leng]
        
        show do
            title "Spin down the cell"
            note "Perform the steps with the following tubes: #{op_out_cell_pellet.to_sentence}."
            check "Balance samples in tubes."
            check "Spin tubes at 4696g for 20 minutes."
            check "While waiting, proceed to measure OD value of reserved 1.5 mL tubes on the rack."
        end
    end
  end
  
  def spin_down_remaining_cell(op_count,op_out_after,op_in_overexpression,op_out_cell_pellet)
    op_table = [["Cell culture ID","225 mL Falcon tube ID"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_overexpression[i]
        row << {content:op_out_cell_pellet[i], check: true}
        op_table << row
    end
    
    run = (op_count/4).floor
    remainder = op_count % 4
    if (remainder) != 0
        run = run + 1
    end
    
    for i in 1..run
        if (i*4 > op_count)
            leng = remainder
        else
            leng = 4
        end
        tube_id = op_out_after[(i-1)*4,leng]
        
        show do 
            title "Continue to harvesting the remaining cell in 2L flask"
            note "Perform the steps with the tubes: #{op_out_cell_pellet.to_sentence}."
            check "Remove tubes from 4°C centrifuge after the first 20-min spin is done."
            check "Pour supernatant into a liquid waste container or sink. Keep pellet."
            check "Pour approximately 220 mL of cell culture into a tube according to the table."
            table op_table
            check "Balance samples in tubes."
            check "Spin tubes at 4696g for 20 minutes." 
        end
        
        show do
            title "Retrieve tubes"
            check "Remove tubes from centrifuge after the 20-min spin is done."
            check "Reset the centrifuge to 23°C."
            check "Pour supernatant into a the used 2L flask or sink. Keep pellet."
            check "Store #{op_out_cell_pellet.to_sentence} in -80°C freezer."
        end
    end
  end
    
  def clean_up
    show do
        title "Clean up"
        check "Place used 2L flask in the dishwashing area."
        check "Return ice bucket."
        check "Clean plastic cuvettes by rinsing with water twice."
        check "Rinse cuvettes with 70% ethanol."
        check "Return them back to the storage box."
    end
  end

end

```
