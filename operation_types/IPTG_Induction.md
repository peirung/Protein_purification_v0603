# IPTG Induction

The objective of this protocol is to use IPTG to induce the expression of a protein of interest.

- Workflow:
  - Make Overexpression > *IPTG Induction* > Harvest Cell
- Steps:
  - [01] Retrieve the 450 mL cell culture from 37°C shaker.
  - [02] Reserve 1 mL of cell culture to a 1.5 mL tube.
  - [03] Add IPTG to the 450 mL cell culture and return it to 37°C shaker.
  - [04] Incubate the cell culture for 4 hours.
  - [05] While cells incubate, measure and record OD600 value of the 1 mL reserved cell culture.
  - [06] Spin down cell and store the cell pellet at -20°C.
### Inputs


- **Overexpression** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "TB Overnight of Plasmid")'>TB Overnight of Plasmid</a>



### Outputs


- **Overexpression** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "TB Overnight (400 mL) of Plasmid")'>TB Overnight (400 mL) of Plasmid</a>

- **Before IPTG** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Overexpression of Plasmid")'>Overexpression of Plasmid</a>

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
    op_out_overexpression = []
    op_out_before = []
    op_count = 0
    operations.running.each do |op|
        op_count = op_count + 1
        op_in_overexpression << op.input("Overexpression").item.id
        op_out_overexpression << op.output("Overexpression").item.id
        op_out_before << op.output("Before IPTG").item.id
    end
    
    # Reserve 1ml cell culture before IPTG induction in Eppendorf with design ID
    reserve_culture_before_iptg(op_count,op_in_overexpression,op_out_before)
    
    # Add isopropyl-1-thio-beta-D-galactopyranoside (IPTG) into the culture.
    add_iptg(op_count,op_in_overexpression)
    
    
    relabel_overexpression(op_count,op_in_overexpression,op_out_overexpression)
        
    # Keep the culture in 30 °C shaker for 5 hours.
    culture_in_shaker_for_4hrs
    
    spin_down_pellet(op_out_before)

    operations.running.each do |op|
        op.set_input_data("Overexpression", :od_value, Random.rand(0.8..1.0)) if debug
        
        od_input = op.input_data("Overexpression", :od_value).to_f
        sample_before = op.output("Before IPTG").item
        sample_before.associate :od_value, od_input
    end
    
    operations.running.each do |op|
      op.output("Overexpression").child_item.move "30°C shaker incubator"
    end
    
    operations.store(io: "output", interactive: true, method: "boxes")
    
    get_protocol_feedback

    return {}
  end

  def reserve_culture_before_iptg(op_count,op_in_overexpression,op_out_before)
    op_table = [["Cell culture ID","Volume","1.5 mL tube ID"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_overexpression[i]
        row << "1 mL"
        row << {content:op_out_before[i], check: true}
        op_table << row
    end
    show do
      title "Reserve 1 mL of cell culture before performing IPTG induction"
      check "Grab <b>#{op_count}</b> 1.5mL tube(s)."
      check "Label tube(s) with ID: #{op_out_before.to_sentence}."
      check "Spray a P1000 pipettor with ethanol and wipe it dry."
      check "Swirl the bottle and transfer 1 mL of cell culture to the corresponding 1.5 mL tube."
      note "**The flask walls should not be touching the pipettor.**"
      check "Place 1.5 mL sample tubes aside for later task."
      table op_table
    end
  end
  

  def add_iptg(op_count,op_in_overexpression)
    op_table = [["Cell culture ID","IPTG Volume"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_overexpression[i]
        row << {content: "1 aliquot", check: true}
        op_table << row
    end
    show do 
      title "IPTG induction"
      check "Grab <b>#{op_count}</b> IPTG aliquot(s) from a box labeled <b>protein purification</b> in -20°C freezer (B1-165)."
      check "Defrost the aliquot(s) by hands or a vortex."
      check "Add one IPTG aliquot to each 2L flask: #{op_in_overexpression.to_sentence}." 
      table op_table
    end
  end
  
  def relabel_overexpression(op_count,op_in_overexpression,op_out_overexpression)
    op_table = [["Old sample ID","New Label"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_overexpression[i]
        row << {content:op_out_overexpression[i], check: true}
        op_table << row
    end
    show do
        title "Relabel 2L flasks with new ID"
        check "Write new ID on a piece of lab tape and affix it on the old label (or cross old ID out and then label with new ID)."
        table op_table
    end
  end
  
  def culture_in_shaker_for_4hrs
    show do 
      title "Incubate"
      check "Place the large cell culture at 30°C shaker incubator for 5 hours."
      bullet "<a href=\'https://www.google.com/search?q=5%20hour%20timer\' target=\'_blank\'>Use a 5 hour Google timer</a> to set a reminder, at which point you will start the <b>\'Harvest Cell\'</b> protocol."
      check "While the cell cultures incubate, finish this protocol by completing the remaining tasks."
    end
  end
  
  def spin_down_pellet(op_out_before)
    show do
        title "Spin down cell"
        note "Perform the steps with the following 1.5 mL tubes: #{op_out_before.to_sentence}."
        check "Spin down at 8000g for 3 minutes."
        check "Remove tubes from the centrifuge. Pipette out the supernatant and keep pellet."
    end
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
        bullet "TB aliquot"
        bullet "a marker pen"
        bullet "sample in a 1.5 mL tube: #{meas_sample.to_sentence}"
    end
    
    show do
        title "Measure OD600 value of cell culture in MolES lab"
        bullet "Open Nanodrop 2000 <b>></b> Home <b>></b> cell culture <b>></b> use cuvette <b>></b> pathlength: 10mm <b>></b> stir speed: off"
        bullet "Blank with 1 mL TB in a cuvette."
        bullet "Discard TB to the liquid waste container."
        bullet "Transfer 1 mL of #{meas_sample.to_sentence} to a new cuvette."
        bullet "Measure OD600 value and record it on the 1.5 mL tube wall."
        bullet "Transfer sample from the cuvette back to its 1.5 mL tube."
        warning "In this step, be careful not to pour samples into the liquid waste container. You will need to keep the samples for later task."
        image "Actions/ProteinPurification/cuvette_nanodrop.jpg"
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
  
end
```
