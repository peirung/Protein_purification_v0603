# Make Overexpression

The objective of this protocol is to grow the cell culture for making a protein of interest.

- Workflow:
  - Make a Starter > *Make Overexpression* > IPTG induction
- Steps:
  - [01] Label a 2L flask and load 450 mL of LB with ampicilin to the flask.
  - [02] Transfer 10 mL of overnight starter culture into the flask.
  - [03] Grow cell culture in 37C shaker for 2.5 hours.
  - [04] Measure OD600 value and make sure the cell density is in the range of 0.6-1.2.
### Inputs


- **Overnight** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "TB Overnight of Plasmid")'>TB Overnight of Plasmid</a>



### Outputs


- **Overexpression** [P]  
  - <a href='#' onclick='easy_select("Sample Types", "Plasmid")'>Plasmid</a> / <a href='#' onclick='easy_select("Containers", "TB Overnight of Plasmid")'>TB Overnight of Plasmid</a>

### Precondition <a href='#' id='precondition'>[hide]</a>
```ruby
def precondition(_op)
  true
end
```

### Protocol Code <a href='#' id='protocol'>[hide]</a>
```ruby
# Author: Pei Wu , Sep 2018

needs "Cloning Libs/Cloning"
needs "Standard Libs/Debug"
needs "Standard Libs/Feedback"

class Protocol

  include Feedback
  include Cloning
  include Debug

  def main
    # Remove the starter from the shaker.
    operations.retrieve
    
    # check the starter(input) have growth
    verify_growth(operations)
    
    operations.make

    op_in_overnight = []
    op_in_bac = []
    op_out_overexpression = []
    op_count = 0

    operations.each do |op|
        op_count = op_count + 1
        op_in_overnight << op.input("Overnight").item.id
        op_in_bac << op.input("Overnight").child_sample.properties["Bacterial Marker"].upcase
        op_out_overexpression << op.output("Overexpression").item.id
    end


    take_flask_add_medium(op_count,op_out_overexpression,op_in_bac)

    grow_culture_in_shaker(op_count,op_in_overnight,op_out_overexpression)

    operations.running.each do |op|
      op.input("Overnight").child_item.store
    end    
    operations.store(io: "input", interactive: true)
    
    set_timer_150mins(op_out_overexpression,op_count)

    od_tmp = measure_od_value(op_out_overexpression)

    record_od_and_decide(od_tmp, operations)

    # Check if there's any sample need to re-measure OD, max. 10 times of re-measureing.
    for i in 1..10
        re_measure_op = operations.select { |op| op.temporary[:redo]==true }.map
        if re_measure_op.size == 0
            break
        else
            op_tmp_overexpression = []
            re_measure_op.each do |op|
                op_tmp_overexpression << op.output("Overexpression").item.id
            end
            od_tmp = measure_od_value(op_tmp_overexpression)
            record_od_and_decide(od_tmp, re_measure_op)
        end
    end
    
    # after 10 times, delete the sample whose OD value is still too low.
    re_measure_op = operations.select { |op| op.temporary[:redo]==true }.map
    if re_measure_op.size != 0
        tmp_id = []
        re_measure_op.each do |op|
            culture = op.output("Overexpression").item
            tmp_id << culture.id
            culture.mark_as_deleted
            culture.save
            op.temporary[:redo] = false
            op.temporary[:delete] = true
            op.error :od_error, "The cell density of #{culture.id} is too high to do induction. Tell the manager and discard the cell culture in the dishwashing area."
        end
        show do
            title "Sample deleted!"
            note "#{tmp_id.to_sentence} are deleted because their OD values are still too low after 10 times."
        end
    end
        

    
    # move overexpression to "37°C shaker incubator"
    operations.running.each do |op|
      op.output("Overexpression").child_item.move "4°C or on bench for (next) protocol"
    end
    
    clean_up(op_out_overexpression)
    
    operations.store(io: "output", interactive: true)
    
    get_protocol_feedback
    
    return {}
  end
  
  def verify_growth(operations)
    verify_growth = show do
        title "Check if overnight starters have growth"
        note "Choose No for the overnight starter that does not have growth. Empty flask and put in the clean station."
        operations.each do |op|
            in_id = op.input("Overnight").item.id
            select ["Yes", "No"], var: "verify #{in_id}", label: "Does flask #{in_id} have growth?"  
        end
    end
    
    operations.select { |op| verify_growth[:"verify#{op.input("Overnight").item.id}".to_sym] == "No" }.each do |op|
        on = op.input("Overnight").item
        on.mark_as_deleted
        on.save
            
        op.error :no_growth, "Your overnight starter had no growth!"
    end
  end    
  
  def take_flask_add_medium(op_count,op_out_overexpression,op_in_bac)
    op_table = [["Overexpression flask ID","Media"]]
    for i in 0..(op_count-1)
      row = []
      row << {content:op_out_overexpression[i], check: true}
      row << "440 mL TB + " + op_in_bac[i]
      op_table << row
    end
    
    show do 
      title "Label flasks and load media"
      check "Grab <b>#{op_count}</b> 2L flask(s) (Sterile). Write ID #{op_out_overexpression.to_sentence} on a piece of lab tape and affix it on the flask."
      check "Grab a graduated cylinder (Sterile). Add 440 mL of media to each flask according to the table below."
      table op_table
      image "Actions/ProteinPurification/glass_flask_column_packing.jpg"
    end
  end
  
   # tell technicians to return the cultures to the 37°C shaker
  def grow_culture_in_shaker(op_count,op_in_overnight,op_out_overexpression)
    op_table = [["Overnight starter ID","Volume","Overexpression flask ID"]]
    for i in 0..(op_count-1)
      row = []
      row << op_in_overnight[i]
      row << "10 mL"
      row << {content:op_out_overexpression[i], check: true}
      op_table << row
    end
    
    show do
      title "Transfer the overnight starter into the flask"
      check "Grab a motorized pipet."
      check "Swirl the bottle and transfer 10 mL of overnight starter culture into the corresponding flask by using a motorized pipet."
      note "**Use a new serological pipette for each batch to avoid cross contamination.**"
      table op_table
      check "Return the motorized pipet."
    end
    
    show do
        title "Store overnight starter"
        check "In the Media Bay, collect <b>#{op_count}</b> 14 mL tube(s)."
        check "Label tube(s) with ID: #{op_in_overnight.to_sentence}."
        check "Pour all the remaining overnight starter in the glass flask to the corresponding 14 mL tube."
    end
  end
  
   def set_timer_150mins(op_out_overexpression,op_count)
    show do
      title "Grow cell culture in 37°C shaker"
      check "Incubate #{op_out_overexpression.to_sentence} at 37°C shaker."
      check "Set a 80-minute timer." 
      timer initial: { hours: 1, minutes: 20, seconds: 0}
    end
    
    show do
        title "Check cell density"
        check "Retrieve #{op_out_overexpression.to_sentence} from 37°C shaker."
        check "Grab <b>#{op_count}</b> 1.5 mL tube(s). Label with ID: #{op_out_overexpression.to_sentence}."
        check "Spray a P1000 pipettor with ethanol and wipe it dry with a paper towel."
        check "Swirl the bottle and transfer 1mL of cell culture to the corresponding 1.5 mL tube."
        note "**The flask walls should not be touching the pipettor.**"
        check "Temporarily keep the 2L flask(s) on bench."
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
        bullet "1 mL of TB aliquot"
        bullet "a marker pen"
        bullet "1 mL of sample in a 1.5 mL tube: #{meas_sample.to_sentence}"
    end
    
    show do
        title "Measure OD600 value of cell culture in MolES lab"
        note "Perform the following steps with #{meas_sample.to_sentence}."
        bullet "Open Nanodrop 2000 <b>></b> Home <b>></b> cell culture <b>></b> use cuvette <b>></b> pathlength: 10mm <b>></b> stir speed: off"
        bullet "Blank with 1 mL TB in a cuvette."
        bullet "Transfer TB back to its 1.5 mL tube."
        bullet "Transfer 1 mL of sample to a new cuvette."
        bullet "Measure OD600 value and record it on the 1.5 mL tube wall."
        bullet "Discard sample into the liquid waste container."
        bullet "Keep the sample tube labeled with OD value."
        image "Actions/ProteinPurification/cuvette_nanodrop.jpg"
    end
    meas_sample.each do |id|
      od = show do
        title "Enter OD value of cell culture"
        get "number", var: "x", label: "OD600 value of #{id}", default: 0.8
      end
      od_value << od[:x]
    end
    return od_value
  end
  
  
  def record_od_and_decide(od_tmp,op_list)
    op_table = [["Batch","Sample ID","OD value","Action"]]

    timer_flag = 0
    sample_remeasure = []
    
    j = 0

    op_list.each do |op|
        row = []
        row << (j + 1)
        culture = op.output("Overexpression").item
        # associate the OD value to the outputs
        culture.associate :od_value, od_tmp[j]
        row << culture.id
        row << od_tmp[j]
        if od_tmp[j].to_f > 3
            culture.mark_as_deleted
            culture.save
            op.temporary[:redo] = false
            op.temporary[:delete] = true
            op.error :od_error, "The cell density of #{culture.id} is too high to do induction. Tell the manager and discard the cell culture in the dishwashing area."
            row << "Discard the sample"
        else
            if od_tmp[j].to_f < 0.8
                row << "Return the cell culture to 37°C shaker and remeasure OD value after 15 minutes."
                op.temporary[:redo] = true
                sample_remeasure << culture.id
                timer_flag = 1
            else
                row << "This cell culture is ready for the next step. If there has another batch of cell culture waiting for a remeasurement, place this cell culture on the bench until all remeasurements are finished." 
                op.temporary[:redo] = false
            end
        end
        op_table << row
        j = j + 1
    end
    
    show do
        title "Cell density and actions"
        table op_table
        if timer_flag == 1
            bullet "Return #{sample_remeasure.to_sentence} to 37°C shaker. The cell density is too low."
            bullet "Remeasure OD value of the sample: #{sample_remeasure.to_sentence} after 15 minutes."
            timer initial: { hours: 0, minutes: 15, seconds: 0}
        end
    end
    
  end
  
  def clean_up(op_out_overexpression)
    show do
        title "Clean up"
        check "Discard used 1.5 mL tube(s) of #{op_out_overexpression.to_sentence}."
        check "Clean plastic cuvettes by rinsing with water twice."
        check "Rinse cuvettes with 70% ethanol."
        check "Return them back to the storage box."
    end
  end

end
```
