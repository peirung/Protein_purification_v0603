# His-tagged Purification

The objective of this protocol is to capture a target protein by using its binding capacity to Ni-NTA resin.

- Workflow:
  - Lyse cell > *His-tagged Purification* > Concentrate Protein 
- Steps:
  - [01] Prepare wash buffer and elution buffer.
  - [02] Remove cell particles from the sample by loading sample to an empty column and passing through an 30 µm filter in the column.
  - [03] Apply sample to the packed column.
  - [04] Wash the column by wash buffer.
  - [05] Elute protein sample by elution buffer.
### Inputs


- **Cell Lysate** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Cell Lysate of Protein Expression")'>Cell Lysate of Protein Expression</a>

- **Resin** [R]  
  - <a href='#' onclick='easy_select("Sample Types", "Resin")'>Resin</a> / <a href='#' onclick='easy_select("Containers", "Column")'>Column</a>



### Outputs


- **Protein** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Protein Elution")'>Protein Elution</a>

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

    op_in_cell_lysate = []
    op_in_resin = []
    op_out_protein = []
    op_wash_amount = []
    op_count = 0
    operations.running.each do |op|
        op_count = op_count + 1
        op_in_cell_lysate << op.input("Cell Lysate").item.id
        op_in_resin << op.input("Resin").item.id
        op_out_protein << op.output("Protein").item.id
        op_wash_amount << op.output("Protein").sample.properties["Wash_Buffer"].to_f
    end
    
    # Turn on the stopcock and let the binding buffer flow through of the column.
    turn_on_stopcock_binding(op_count,op_in_resin,op_in_cell_lysate)
        
    # Apply the supernatant with design ID into the gravity flow column with resin in it.
    apply_supernatnat(op_count,op_in_cell_lysate,op_in_resin)
    
    reload_sample(op_count,op_in_resin,op_out_protein,op_in_cell_lysate)
    
    apply_supernatnat(op_count,op_in_cell_lysate,op_in_resin)
    
    #turn_on_stopcock_supernatant(op_count,op_in_cell_lysate,op_in_resin)
    buffer_preparation(op_count,op_in_cell_lysate,op_wash_amount,op_out_protein)
    
    # Add 50mL wash buffer to the column and let it flow through the column completely. Then turn off the stopcock.
    add_wash_buff(op_count,op_in_resin,op_out_protein,op_in_cell_lysate)
    
    prepare_elution_buffer(op_count,op_out_protein,op_in_cell_lysate)
        
    # Add 7.5mL elution buffer into the column.
    add_elution_buff(op_count,op_in_resin,op_out_protein)
        
    operations.running.each do |op|
      op.output("Protein").child_item.move "4°C or on ice for the (next) protocol"
    end

    operations.store(io: "output", interactive: true)
    
    clean_up
    
    get_protocol_feedback

    return {}

  end
  
  # approximate final concentration of imidazole: 5-30mM wash buffer/250mM elution buffer
  def buffer_preparation(op_count,op_in_cell_lysate,op_wash_amount, op_out_protein)
    op_table =[["Falcon tube ID","Purification buffer volume","Imidazole volume"]]
    
    for i in 0..(op_count-1)
        #if !op_wash_amount[i]
            #op_wash_amount[i] = 5
        #end
        row = []
        row << op_in_cell_lysate[i]
        row << {content: "15 mL", check: true}
        row << {content: "#{op_wash_amount[i]*3} µl", check: true}
        op_table << row
    end
    
    show do
        title "Prepare wash buffer"
        check "Grab Purification Buffer from 4°C refrigerator."
        check "Grab a imidazole aliquot, in a box labeled <b>protein purification</b> in -20°C freezer (B1-165)"
        #check "Grab <b>#{op_count}</b> 50 mL Falcon tube and label with #{op_in_cell_lysate.to_sentence} on the tube wall."
        check "Prepare wash buffer according to the table."
        check "Mix the buffer by quick vortexing."
        table op_table
    end
  end

  def turn_on_stopcock_binding(op_count,op_in_resin,op_in_cell_lysate)
    show do
        title "Remove liquid from the column"
        check "Open the column outlet. Allow buffer to flow out of the column."
        check "Leave 0.5-1 mL buffer in the column to prevent resin dry out, then close the column outlet."
        check "Discard flow through in the tube. Return the tube underneath the corresponding column"
        check "Grab <b>#{op_count}</b> disposable dropper(s)."
        check "Label droppers with ID: #{op_in_cell_lysate.to_sentence}"
        check "Place dropper in its corresponding sample tube (as shown in the picture). Place caps aside for later use."
        image "Actions/ProteinPurification/droppers.jpg"
    end
  end
    
  def apply_supernatnat(op_count,op_in_cell_lysate,op_in_resin)
    op_table = [["Sample ID","Column ID"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_cell_lysate[i]
        row << op_in_resin[i]
        op_table << row
    end
    show do
       title "Load sample to a column"
       check "Load sample to its corresponding column by using a plastic dropper."
       note "**Be careful not to disturb the resin. Use a new dropper for each sample to avoid cross contamination.**"
       table op_table
       check "Partially open the column oulet and make sample slowly flow out of the column (as show in the picture)."
       check "Continue loading sample to its corresponding column until all of the sample has applied to the column."
       check "Leave 0.5-1 mL sample in the column to prevent resin dry out, then close the column outlet."
       image "Actions/ProteinPurification/outlet_angle.jpg"
    end
  end
  
  def reload_sample(op_count,op_in_resin,op_out_protein,op_in_cell_lysate)
    op_table = [["Elution tube ID","Sample tube ID"]]
    for i in 0..(op_count-1)
        row = [] 
        row << op_in_resin[i]
        row << op_in_cell_lysate[i]
        op_table << row
    end
    show do
        title "Transfer elution to its sample tube"
        check "Pour all the elution back to its sample tube according to the table." 
        table op_table
        check "Return the empty elution tube underneath its corresponding column for collecting sample."
    end
  end
  
  
  def add_wash_buff(op_count,op_in_resin,op_out_protein,op_in_cell_lysate)
    op_table = [["Wash buffer tube ID","Column ID"]]
    for i in 0..(op_count-1)
        row = []
        row << op_in_cell_lysate[i]
        row << op_in_resin[i]
        op_table << row
    end
    show do
       title "Column wash"
       check "Discard flow through in the tube. Return the tube back underneath the corresponding column."
       check "According to the table, load all of the wash buffer to each column by using a dropper."
       table op_table
       check "Fully open the column outlet. Allow buffer to flow out of the column."
       check "Leave 0.5-1 mL buffer in the column to prevent resin dry out, then close the column outlet."
       check "Remove the tubes underneath each column. Discard flow through, all falcon tubes, and droppers."
    end
  end
  
  def prepare_elution_buffer(op_count,op_out_protein,op_in_cell_lysate)
    op_table = [["Falcon tube ID","Purification buffer volume","Imidazole volume"]]
    for i in 0..(op_count-1)
        row = []
        row << op_out_protein[i]
        row << {content:"7.5 mL", check: true}
        row << {content:"400 µl", check: true}
        op_table << row
    end
        
    show do
        title "Prepare elution buffer"
        check "Grab <b>#{op_count}</b> Falcon tube(s)."
        check "Label tube(s) with ID: #{op_out_protein.to_sentence} on the tube wall and cap."
        check "Prepare elution buffer according to the table."
        check "Mix the buffer by quick vortexing."
        table op_table
    end
  end
  
  def add_elution_buff(op_count,op_in_resin,op_out_protein)
    op_table_1 = [["Elution buffer tube ID","Column ID"]]
    for i in 0..(op_count-1)
        row = []
        row << op_out_protein[i]
        row << op_in_resin[i]
        op_table_1 << row
    end
        
    show do
       title "Elute protein sample"
       check "Grab <b>#{op_count}</b> droppers."
       check "Load elution buffer to its corresponding column according to the table, and then return the empty tube underneath the column for collecting sample."
       table op_table_1
       check "Fully open the column oulet and make sample slowly flow out of the column."
       check "Close the column outlet."
       check "Screw the cap back onto the tube. Keep sample in ice bath."
    end
  end
  
  def clean_up
    show do
        title "Clean up"
        check "Return the purification buffer to 4°C refrigerator."
        check "Return imidazole aliquot to -20°C freezer."
        check "Detach adaptors from columns."
        check "Clean up adaptors by rinsing with water."
        check "Discard used columns and Falcon tubes."
        check "Discard used droppers."
        check "Disassemble the support stand and return it to the drawer."
    end
  end
  
end
```
