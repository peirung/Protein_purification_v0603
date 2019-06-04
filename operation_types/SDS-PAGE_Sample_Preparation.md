# SDS-PAGE Sample Preparation

The objective of this protocol is to prepare samples for SDS-PAGE analysis.

- Workflow:
  - Concentrate Protein > *SDS-PAGE Sample Preparation* > SDS-PAGE Analysis
- Steps:
  - [01] Grab all reserved samples: cell pellet before IPTG induction, cell pellet after IPTG induction and concentrated protein sample.
  - [02] Add lysis buffer to first two samples and vortex to mix well.
  - [03] Add sample buffer to all samples.
  - [04] Heat at 95°C for 5 minutes.
### Inputs


- **Before IPTG** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Overexpression of Plasmid")'>Overexpression of Plasmid</a>

- **After IPTG** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "Overexpression of Plasmid")'>Overexpression of Plasmid</a>

- **Protein** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Concentrated Protein")'>Concentrated Protein</a>



### Outputs


- **Before IPTG** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Laemmli Sample Buffer")'>Laemmli Sample Buffer</a>

- **After IPTG** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Laemmli Sample Buffer")'>Laemmli Sample Buffer</a>

- **Protein** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Laemmli Sample Buffer")'>Laemmli Sample Buffer</a>

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
    
    # Gather all reserved sample: 
    # - sample 01: before IPTG induction with design ID
    # - sample 02: after IPTG induction with design ID
    # - sample 03: purified protein
    
    op_in_protein   = []
    op_in_before    = []
    op_in_after     = []
    op_out_protein  = []
    op_out_before   = []
    op_out_after    = []
    
    lysis_before      = []
    lysis_after       = []
    amount_protein    = []
    
    op_count = 0
    
    operations.running.each do |op|
        op.set_input_data("Before IPTG", :od_value, Random.rand(0.8..1.0)) if debug
        op.set_input_data("After IPTG", :od_value, Random.rand(0.8..1.0)) if debug
        op.set_input_data("Protein", :protein_concentration, Random.rand(0.01..3)) if debug

        op_count = op_count + 1
        op_in_protein   << op.input("Protein").item.id
        op_in_before    << op.input("Before IPTG").item.id
        op_in_after     << op.input("After IPTG").item.id
        op_out_protein  << op.output("Protein").item.id
        op_out_before   << op.output("Before IPTG").item.id
        op_out_after    << op.output("After IPTG").item.id
        
        tmp_before  = (op.input_data("Before IPTG", :od_value).to_f * 100).floor
        tmp_after   = (op.input_data("After IPTG", :od_value).to_f * 100).floor
        # to get 0.01mg from sample, the amount is 0.01(mg)*1000(ul/mL)/protein_concetration(mg/mL)
        tmp_protein = (20*3.5/(op.input_data("Protein", :protein_concentration).to_f)).round(1) # ul
        
        # Record the input (Concentrated Protein) protein concentration to output (Laemmli Sample Buffer).
        protein_out = op.output("Protein").item
        protein_out.associate :protein_concentration, op.input_data("Protein", :protein_concentration).to_f
        
        lysis_before   << tmp_before
        lysis_after    << tmp_after
        amount_protein  << tmp_protein  # ul
        
    end

    # Preheat the heat plate to 98°C.
    sample_preparation(op_in_protein,op_in_before,op_in_after)
    
    add_lysis(op_count,op_in_before,op_in_after,lysis_before,lysis_after)
    
    grab_tubes(op_count,op_in_before,op_in_after,op_in_protein,op_out_protein,op_out_before,op_out_after,amount_protein)
    
    operations.each do |op|
        op.input("Before IPTG").item.mark_as_deleted
        op.input("After IPTG").item.mark_as_deleted
    end
    
    operations.store
    
    clean_up
     
    get_protocol_feedback

    return {}

  end

  def sample_preparation(op_in_protein,op_in_before,op_in_after)
    show do
        title "Set up a heat block"
        check "Set a heat block to 98°C."
        check "Grab a ice block."
        check "Place #{op_in_protein.to_sentence} on ice block."
    end
  end

  
  def add_lysis(op_count,op_in_before,op_in_after,lysis_before,lysis_after)
    op_table = [["Sample stock ID","Lysis Buffer Volume (µl)"]]
    for i in 0..(op_count-1)
        row = []
        row << {content:op_in_before[i], check: true}
        row << lysis_before[i]
        op_table << row
        row = []
        row << {content:op_in_after[i], check: true}
        row << lysis_after[i]
        op_table << row
    end
    show do
        title "Suspend cell pellet in lysis buffer"
        #"Formula: lysis buffer(uL) = OD value* 100 uL lysis buffer."
        check "Grab the lysis buffer from 4°C refrigerator."
        check "Add lysis buffer to tubes according to the table."
        table op_table
        check "Vortex samples for 10 seconds or until no clumps of cell pellet."
        check "Return the lysis buffer to 4°C refrigerator."
    end
  end
  
  
  def grab_tubes(op_count,op_in_before,op_in_after,op_in_protein,op_out_protein,op_out_before,op_out_after,amount_protein)
    op_table_2 = [["Tube ID","Volume"]]
    add_amount = []
    for i in 0..(op_count-1)
        row = []
        row << {content:(i*3+3), check: true}
        if amount_protein[i] <= 20*3.5
            amount_tmp = 20*3.5 - amount_protein[i]
        else
            amount_tmp = 0
        end
        add_amount << amount_tmp
        row << "#{amount_tmp.ceil} µl"
        op_table_2 << row
    end
    
    show do
        title "Sample Preparation"
        check "Grab <b>#{op_count*3}</b> tube(s) and label with ID from 1 to #{op_count*3}."
        check "Add PBS to tubes according to the table:"
        table op_table_2
    end
    
    op_table = [["Sample ID (Stock)","Volume","Tube ID (Output)"]]
    all_output = []
    for i in 0..(op_count-1)
        # table
        row = []
        row << op_in_before[i]
        row << "70 µl"
        row << (i*3+1)
        op_table << row
        row = []
        row << op_in_after[i]
        row << "70 µl"
        row << (i*3+2)
        op_table << row 
        row = []
        row << op_in_protein[i]
        row << "#{amount_protein[i]} µl"
        row << (i*3+3)
        op_table << row
        # all output IDs
        all_output << op_out_before[i]
        all_output << op_out_after[i]
        all_output << op_out_protein[i]
    end
    
    show do
        title "Add Sample"
        check "Pipette sample to corresponding tubes according to the table."
        table op_table
        check "Retrun #{op_in_protein.to_sentence} on ice block after use."
    end
    
    
    op_table_1 = [["Tube ID","Volume"]]
    all_output = []
    for i in 0..(op_count-1)
        # table
        row = []
        row << {content:(i*3+1), check: true}
        row << "70 µl"
        op_table_1 << row
        row = []
        row << {content:(i*3+2), check: true}
        row << "70 µl"
        op_table_1 << row 
        row = []
        row << {content:(i*3+3), check: true}
        row << "#{(amount_protein[i] + add_amount[i]).ceil} µl"
        op_table_1 << row
        # all output IDs
        all_output << op_out_before[i]
        all_output << op_out_after[i]
        all_output << op_out_protein[i]
    end
    
    show do
    title "Add sample buffer"
        check "Grab sample buffer from 4°C refrigerator (R1-250)."
        check "Pipette sample buffer to each tube according to the table. (Perform this step in fume hood)"
        table op_table_1
        check "Return the sample buffer to the 4°C refrigerator (R1-250)."
    end
    
    show do
       title "Heat samples"
       check "Seal tube caps (ID from 1 to #{op_count*3}) with parafilm, preventing sample from popping out of a tube under boil."
       check "Heat samples for 10 minutes at 98°C."
       timer initial: { hours: 0, minutes: 10, seconds: 0}
       check "Retrieve tubes from the heat block. Place samples on a rack."
       check "Turn off the heat block or set the temperature back to its setting."
       check "Remove the parafilm from the tubes."
    end
    
    op_table_3 = [["Tube ID","Sample ID"]]
    for i in 0..(op_count-1)
        # table
        row = []
        row << (i*3+1)
        row << {content:op_out_before[i], check: true}
        op_table_3 << row
        row = []
        row << (i*3+2)
        row << {content:op_out_after[i], check: true}
        op_table_3 << row 
        row = []
        row << (i*3+3)
        row << {content:op_out_protein[i], check: true}
        op_table_3 << row
    end
    
    show do
       title "Transfer samples"
       check "Spin sample tubes at 17,000 g for 10 minutes."
       check "Grab <b>#{op_count*3}</b> 1.5 mL tubes and label with sample ID according to the table."
       check "Transfer all the supernatant to new tubes according to the table.(Perform this step in fume hood)"
       table op_table_3
       check "Discard the tube labeled with ID from 1 to #{op_count*3}."
       check "Discard the tube labeled with ID:#{op_in_before.to_sentence}, #{op_in_after.to_sentence}"
    end
    
  end

  def clean_up
    show do
        title "Clean up"
        check "Return the ice block."
    end
  end

end
```
