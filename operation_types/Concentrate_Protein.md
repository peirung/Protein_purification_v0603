# Concentrate Protein

The objective of this protocol is to lower the imidazole concentration in the protein storage buffer and concentrate protein sample to a higher concentration which both help with increasing protein stability. 

- Workflow:
  - His-tagged Purification > *concentrate protein* > SDS-PAGE Sample Preparation
- Steps:
  - [01] Grab a Centriprep tube and equilibrate the filter with water.
  - [02] Spin for 10 minutes and pour off water
  - [03] Pour the protein elution to the Centriprep tube.
  - [04] centrifuge and collect concentrated protein sample.
  - [05] Measure and record protein concentration.
  - [06] Add glycerol to protein sample for preventing sample from being frozen in -20°C.
### Inputs


- **Protein** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Protein Elution")'>Protein Elution</a>



### Outputs


- **Protein** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Concentrated Protein")'>Concentrated Protein</a>

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

    op_in_protein = []
    op_out_protein = []
    op_in_protein_leng = []
    op_in_protein_tube = []
    op_count = 0
    operations.running.each do |op|
        # added 3/26 protein_length
        #op.set_input_data("Protein", :Size, Random.rand(10..50)) if debug
        
        tmp_size = op.output("Protein").sample.properties["Size"].to_f
        #tmp_size = Random.rand(10..50) if debug

        op_count = op_count + 1
        op_in_protein << op.input("Protein").item.id
        op_out_protein << op.output("Protein").item.id
        
        # added 3/26 protein_length
        op_in_protein_leng << tmp_size.to_f.round(0)
        
        if tmp_size.to_f >= 40
            op_in_protein_tube << "White Membrane"
        else
            op_in_protein_tube << "Green Membrane"
        end
    end

    # Set the Refrigerated Centrifuge to 4°C first.
    set_refrig_centrifuge_temp
        
    # Take one concentrator and label it with design ID.
    take_and_label_concentrator(op_count,op_in_protein_leng,op_in_protein_tube)
        
    add_sample_to_tube(op_count,op_in_protein)
        
    add_elution_and_centrifuge(op_count,op_in_protein)
    
    buffer_exchange(op_count)
    
    add_elution_and_centrifuge(op_count,op_in_protein)
    
    take_and_label_eppendorf(op_count,op_out_protein)
    
    # Transfer the 1mL sample to the labeled Eppendorf.
    transfer_sample_to_eppendorf(op_count,op_in_protein,op_out_protein)
        
    pro_concentration = measure_concentration(op_out_protein)
        
    # Record the protein concentration.
    i = 0
    operations.running.each do |op|
        protein_out = op.output("Protein").item
        protein_out.associate :protein_concentration, pro_concentration[i]
        i = i + 1
    end
    
    operations.store(io: "output", interactive: true, method: 'boxes')
    
    clean_up
    
    get_protocol_feedback
    
    return {}
  end
  
  def set_refrig_centrifuge_temp
    show do
        title "Pre-cool a centrifuge"
        bullet "Set the centrifuge to 4°C."
        image "Actions/ProteinPurification/centrifuge.jpg"
    end
  end

  def take_and_label_concentrator(op_count,op_in_protein_leng,op_in_protein_tube)
    #added 3/26
    op_table = [["Label","Protein (kDa)","Tubes"]]
    for i in 0..(op_count-1)
        row = []
        row << {content:i+1, check: true}
        row << op_in_protein_leng[i]
        row << "#{op_in_protein_tube[i]} Tube"
        op_table << row
    end
    show do
        title "Label Centriprep tubes"
        check "Grab Centriprep tube(s) (white or green membrane) as follows in the table."
        check "Label Centriprep tube(s) from 1 to #{op_count} on the tube wall and cap."
        table op_table
        image "Actions/ProteinPurification/centriprep_tube.jpg"
    end
  end
    
  def add_sample_to_tube(op_count,op_in_protein)
    op_table = [["Sample ID","Centriprep tube ID"]]
    for i in 0..(op_count-1)
        row = []
        row << {content:op_in_protein[i], check: true}
        row << i+1
        op_table << row
    end
    
    show do
        title "Load protein samples"
        check "Remove caps from Centriprep tubes."
        check "Pour sample into sample container according to the table (as shown in the picture)."
        table op_table
        check "Screw caps back on tubes."
        image "Actions/ProteinPurification/pour_sample_to_concentrator.jpg"
    end
  end
    
  def add_elution_and_centrifuge(op_count,op_in_protein)
    show do
        title "Concentrate protein samples"
        bullet "Perform steps with tubes: ID from 1 to #{op_count}."
        check "Spin at 3000g for 1 hour. Make sure to balance tubes."
        check "Remove Centriprep tubes from the centrifuge."
        check "Discard flow through in the filtrate collector (as shown in the picture)."
        warning "Protein sample is concentrated in the <b>sample container</b>. Be careful not to empty a wrong container."
        check "Reassemble Centriprep tubes."
        image "Actions/ProteinPurification/collect_sample.jpg"
    end
  end

  def buffer_exchange(op_count)
    op_table = [["Centriprep tube ID","1xPBS","50% Glycerol"]]
    for i in 0..(op_count-1)
        row = []
        row << i+1
        row << {content:"500 µl", check: true}
        row << {content:"500 µl", check: true}
        op_table << row
    end

    show do
        title "Buffer exchange"
        bullet "Perform steps with tubes: ID from 1 to #{op_count}."
        check "Grab PBS (pH7.4) "
        check "Add 5 mL PBS into each sample container by using a serological pipette."
        check "Reassemble Centriprep tubes."
        #table op_table
        image "Actions/ProteinPurification/add_pbs_buffer_exchange.jpg"
    end
  end
  
  def take_and_label_eppendorf(op_count,op_out_protein)
    show do
        title "Grab and label tubes"
        check "Grab <b>#{op_count}</b> 1.5 mL tubes."
        check "Label tubes with ID: #{op_out_protein.to_sentence}."
    end
  end
  
 def transfer_sample_to_eppendorf(op_count,op_in_protein,op_out_protein)
    op_table = [["Centriprep tube ID","1.5 mL tube ID"]]
    for i in 0..(op_count-1)
        row = []
        row << i+1
        row << {content:op_out_protein[i], check: true}
        op_table << row
    end
    
    show do
        title "Collect Samples"
        check "Set a P1000 pipettor to 1000 µl."
        check "Check on sample volume :"
        bullet "If sample volume is equal or less than 1000 µl, transfer sample to corresponding 1.5 mL tube according to the table. Then keep sample in ice bath."
        bullet "If sample volume is more than 1000 µl, don't withdraw the sample. Spin the Centriperp tube at 3000g for another 15 
        minutes or until sample volume reaches 1000 µl."
        table op_table
        image "Actions/ProteinPurification/check_on_volume.jpg"
    end
  end
  
  def measure_concentration(op_out_protein)
    protein_concentration = []
    
    op_out_protein.each do |id|
        concetration = show do
            title "Measure protein concentration"
            check "Open Nanodrop in protein mode. Blank with 2 µl PBS."
            check "Nanodrop 2 µl protein sample."
            check "Enter protein concentration."
            get "number", var: "x", label: "Protein concetration of #{id}", default: 0
        end
        protein_concentration << concetration[:x]
    end
    return protein_concentration
  end
  
  def clean_up
    show do
        title "Clean up"
        check "Reset the centrifuge to 23°C."
        check "Discard used Centriprep tubes."
        check "Return the ice bucket/ice block."
    end
  end
end

```
