# Make a Starter

The objective of this protocol is to prepare an overnight starter for the next day to amplify a large volume of cell culture.

- Workflow:
  - Check Plate > *Make a Starter* > Make Overexpression 
- Steps:
  - [01] Grab a 200 mL glass flask and label with ID.
  - [02] Load 12 mL of LB with ampicilin to the flask.
  - [03] Use a 10 µl sterile tip to inoculate an isolated colony from the plate into 200 mL glass flask.
  - [04] Incubate cell in 37C shaker overnight.
### Inputs


- **Plasmid** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Checked E coli Plate of Plasmid")'>Checked E coli Plate of Plasmid</a>
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Plasmid Glycerol Stock")'>Plasmid Glycerol Stock</a>



### Outputs


- **Overnight** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "TB Overnight of Plasmid")'>TB Overnight of Plasmid</a>

### Precondition <a href='#' id='precondition'>[hide]</a>
```ruby
def precondition(_op)
  true
end
```

### Protocol Code <a href='#' id='protocol'>[hide]</a>
```ruby
# Author: Pei Wu , Oct 2018
# Note: modified from "Cloning/Make Overnight Suspension"

needs "Cloning Libs/Cloning"
needs "Standard Libs/Debug"
needs "Standard Libs/Feedback"

class Protocol

  include Feedback    
  include Cloning
  include Debug

  def main
      
    operations.retrieve(interactive: false)
    
    operations.make
    
    p_ot = ObjectType.where(name: "Checked E coli Plate of Plasmid").first 
    
    raise "Could not find object type 'Checked E coli Plate of Plasmid'" unless p_ot
    
    plate_inputs = operations.running.select { |op| op.input("Plasmid").item.object_type_id == p_ot.id }
    
    g_ot = ObjectType.where(name: "Plasmid Glycerol Stock").first 
    
    raise "Could not find object type 'Plasmid Glycerol Stock'" unless g_ot 
    
    glycerol_stock_inputs = operations.running.select { |op| op.input("Plasmid").item.object_type_id == g_ot.id }
    
    overnight_steps plate_inputs, "Checked E coli Plate of Plasmid" if plate_inputs.any?
    overnight_steps glycerol_stock_inputs, "Plasmid Glycerol Stock" if glycerol_stock_inputs.any?
    
    # Associate input id with from data for overnight.
    operations.running.each do |op|
      gs = op.input("Plasmid").item
      on = op.output("Overnight").item
      
      on.associate :from, gs.id
      pass_data "sequencing results", "sequence_verified", from: gs, to: on
    end
    
    operations.running.each do |op|
      op.output("Overnight").child_item.move "37°C shaker incubator"
    end
    
    operations.store
    
    get_protocol_feedback
    
    return {}

  end 
  
  def overnight_steps(ops, ot)
    if ot == "Plasmid Glycerol Stock"
      ops.retrieve interactive: false
    else
      ops.retrieve
    end
    
    # Sorting ops by the bacterial marker attribute
    temp = ops.sort do |op1,op2|
      op1.input("Plasmid").child_sample.properties["Bacterial Marker"].upcase <=> op2.input("Plasmid").child_sample.properties["Bacterial Marker"].upcase
    end
    ops = temp
    
    ops.extend(OperationList)
   
    #Label and load overnight tubes 
    label_load_tubes ops

    #Inoculation
    inoculate ot, ops
      
  end
  
  # Given operations, tells the technician to label and load the tubes the tubes
  def label_load_tubes ops
    show do
      title "Label and load overnight glass flasks"
      check "Grab <b>#{ops.length}</b> 125 mL glass flask (sterile)."
      check "Grab media in 4°C refrigerator according to the table below."
      check "Write the overnight ID on a piece of lab tape and affix it on the glass flask."
      check "Remove caps from the top of glass flasks."
      check "Add 15 mL of media to each glass flask by using a motorized pipet."
      table ops.start_table
        .output_item("Overnight", checkable: true)
        .custom_column(heading: "Media") { |op| "TB+" + op.input("Plasmid").child_sample.properties["Bacterial Marker"].upcase}
        .custom_column(heading: "Volume") { |op| "15 mL" }
        .end_table
    end
  end
  
  # Tells the technician to inoculate colonies from plate into glass flask.
  def inoculate ot, ops
    show {
      title "Inoculation from #{ot}"
      # note "Use 10 uL sterile tips to inoculate colonies from plate into 14 mL tubes according to the following table." if ot == "Checked E coli Plate of Plasmid"
      check "Spray a P1000 pipettor with ethanol and wipe it dry."
      check "Use 1000 µl pipette tip to inoculate an isolated colony from plate into 125 mL flask according to the following table." if ot == "Checked E coli Plate of Plasmid"
      check "Use 100 µl pipette to inoculate cells from glycerol stock into the 125 mL flask according to the following table." if ot == "Plasmid Glycerol Stock"
      note "**The inside flask walls should not be touching the pipettor.**"
      check "Put caps back on the top of glass flasks."
      check "Return the media to the refrigerator."
      table ops.start_table
        .input_item("Plasmid", heading: ot)
        .custom_column(heading: "#{ot} Location") { |op| op.input("Plasmid").item.location }
        .output_item("Overnight", checkable: true)
        .end_table
     
    } 
  end
end 
```
