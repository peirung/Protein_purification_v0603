# Column Packing

The objective of this protocol is to prepare a column packed with Ni-NTA resin and pre-equilibrate with binding buffer.

- Workflow:
  - Harvest Cell > *Column Packing* > Lyse Cell 
- Steps:
  - [01] Prepare binding buffer.
  - [02] Grab a column and attach an adaptor to it.
  - [03] Clamp the column to a support stand.
  - [04] Load Ni-NTA resin into the column.
  - [05] Equilibrate Resin by the binding buffer.




### Outputs


- **Affinity Column** [B]  
  - <a href='#' onclick='easy_select("Sample Types", "Resin")'>Resin</a> / <a href='#' onclick='easy_select("Containers", "Column")'>Column</a>

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
    
    op_out_column = []
    op_count = 0

    operations.each do |op|
        op_count = op_count + 1
        op_out_column << op.output("Affinity Column").item.id
    end
    
    tube_count = (op_count*20/50).floor
        if remainer = (op_count*20 % 50) != 0
            tube_count = tube_count + 1
        end
    
    label_column(op_count,op_out_column)
        
    put_column_onto_stand
        
    mix_resin(op_count, op_out_column)
        
    load_20ml_binding_buffer(op_out_column)
    
    clean_up
    
    operations.store
    
    get_protocol_feedback

    return {}
  end
  
  #prepare binding buffer (30mM imidazole_final concentration)
  #20mL/per column, imidazole aliquot concentration: 5M
  def prepare_binding_buffer(op_count,tube_count)
    op_table = [["Item","Volume"],
    ["Purification buffer",{content:"#{op_count*23} mL", check: true}],
    ["Imidazole aliquot",{content:"#{op_count*138} µl", check: true}]
    ]
    show do
        title "Prepare binding buffer"
        check "Grab a beaker."
        note "Write <b>Binding buffer</b> on a piece of lab tape and affix it on the beaker."
        bullet "Grab the following items:"
        note "Purification buffer, in the Media Bay"
        note "Imidazole aliquot, in a box labeled <b>Protein purification</b> in -20°C freezer (B1-165)"
        bullet "Prepare binding buffer according to the table."
        table op_table
        note "Return the buffer and aliquot to the Media Bay and storage box."
        #image "Actions/ProteinPurification/purification_buffer.jpg"
    end
  end

  def label_column(op_count,op_out_column)
    show do 
      title "Grab and label columns"
      check "Grab <b>#{op_count}</b> column(s) and label the column with ID: #{op_out_column.to_sentence}."
      check "Grab <b>#{op_count}</b> flow adaptor(s)."
      check "Remove the snap-off tip at the bottom of the column. Attach a flow adaptor to it (as shown in the picture)."
      image "Actions/ProteinPurification/empty_column_column_packing.jpg"
    end
    
    show do
        title "Column preparation"
        check "Close the column outlet."
        image "Actions/ProteinPurification/oulet_column_packing.jpg"
    end
  end
  
  def put_column_onto_stand
    show do
       title "Column preparation"
       check "Grab a support stand and clamps. Clamp a column to the support stand."
       check "If you are packing more than two columns, grab tubes racks and a storage box to set up a support stand for columns (as shown in the picture)"
       image "Actions/ProteinPurification/packing_four_columns.jpg"
    end
  end
  
  def mix_resin(op_count, op_out_column)
    op_table = [["Column ID","Bed volume of Ni-NTA resin"]]
    for i in 0..(op_count-1)
        row = []
        row << {content:op_out_column[i], check: true}
        row << "2 mL"
        op_table << row
    end
    show do
       title "Load Ni-NTA resin"
       note "Perform the steps with the columns: #{op_out_column.to_sentence}."
       check "Grab <b>#{op_count}</b> 50 mL Falcon tubes and label with #{op_out_column.to_sentence}."
       check "Place the tubes underneath the corresponding column."
       check "Grab Ni-NTA resin from 4°C refrigerator (R1-250). "
       check "Pipet up and down to homogenize the resin by using a P1000 pipettor."
       check "Load 2 mL of Ni-NTA slurry to each column."
       check "Open the column outlet and let the liquid flow through the column."
       check "Check on the resin volume in the column. The volume should be betweeen 1.0-1.5mL mark on the column."
       check "Add more resin to the column if the volume is less than 1 mL."
       check "Return the resin to 4°C refrigerator (R1-250)."
       image "Actions/ProteinPurification/resin_column_packing.jpg"
    end
  end
  
  def load_20ml_binding_buffer(op_out_column)
    show do
       title "Equilibrate Resin"
       note "Perform the steps with the columns: #{op_out_column.to_sentence}."
       check "Close the column outlet."
       check "Grab the purification buffer from 4°C refrigerator."
       check "Grab a motorized pipet and 25 mL serological pipette."
       check "Drip 20 mL of purification buffer to each column along the column wall."
    end
  end
  
  def clean_up
    show do
        title "Clean up"
        check "Return the purification buffer to 4°C refrigerator."
        check "Return the motorized pipet and discard the used serological pipette."
    end
  end
  
  
end
```
