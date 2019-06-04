# SDS-PAGE Analysis

The objective of this protocol is to analyze proteins in a sample by electrophoresis. 

- Workflow:
  - SDS-PAGE Gel Casting > *SDS-PAGE Analysis* > Scan Gel
- Steps:
  - [01] Grab the samples which are prepared by the protocol "SDS-PAGE Sample Preparation".
  - [02] Load samples into wells using a loading tip.
    
| Well 01       | Well 02              | Well 03             | Well 04                              |
| --------------| ---------------------| --------------------| -------------------------------------|
| Protein ladder| Sample 01            | Sample 02           | Sample 03                            |
|               | Before IPTG induction| After IPTG induction| Protein Sample (Concentrated protein)|
    
  - [03] Attach the lid to the tank and start electrophoresis.
  - [04] Run gel until the protein ladder is fully separated.
  - [05] Remove gel from the glass cassette and wash the gel for three times with clean water.
  - [06] Stain gel with gel staining buffer for 1 hour.
  - [07] Destain gel with clean until the gel matrix background is clear.
### Inputs


- **Before IPTG** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Laemmli Sample Buffer")'>Laemmli Sample Buffer</a>

- **After IPTG** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Laemmli Sample Buffer")'>Laemmli Sample Buffer</a>

- **Protein** [PR]  
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "Laemmli Sample Buffer")'>Laemmli Sample Buffer</a>

- **SDS Gel** [G]  Part of collection
  - NO SAMPLE TYPE / <a href='#' onclick='easy_select("Containers", "12% SDS Gel in Tank")'>12% SDS Gel in Tank</a>



### Outputs


- **SDS Gel** [PR]  Part of collection
  - <a href='#' onclick='easy_select("Sample Types", "Protein")'>Protein</a> / <a href='#' onclick='easy_select("Containers", "12% SDS Gel in Coomassie Staining")'>12% SDS Gel in Coomassie Staining</a>

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
    
    operations.retrieve

    op_in_protein   = []
    op_in_before    = []
    op_in_after     = []
    
    amount_protein  = []
    
    op_count = 0
    
    operations.running.each do |op|
        op.set_input_data("Protein", :protein_concentration, Random.rand(0.1..3)) if debug
        # 
        op_count = op_count + 1
        op_in_protein   << op.input("Protein").item.id
        op_in_before    << op.input("Before IPTG").item.id
        op_in_after     << op.input("After IPTG").item.id
        

        tmp_protein = (2*20/(op.input_data("Protein", :protein_concentration).to_f)).ceil # ul
        if tmp_protein <= 40
            amount_protein << 40
        elsif tmp_protein <= 60
            amount_protein << tmp_protein
        else
            amount_protein << 60
        end
        
    end

    op_in_gel = operations.map { |op| op.input("SDS Gel").collection.id }.uniq
    
    # Don't use generic operations.make
    operations.each do |op|
        op.output("SDS Gel").make_part(
          op.input("SDS Gel").collection,
          op.input("SDS Gel").row,
          op.input("SDS Gel").column
        )
    end
    
    op_out_gel = operations.map { |op| op.output("SDS Gel").collection.id }.uniq
    
    tank_count = (op_in_gel.size)/4.floor
    remainder = (op_in_gel.size) % 4
    if remainder != 0
        tank_count = tank_count + 1
    end

    # Load samples into wells using a pipet with gel loading tips. [note] load samples slowly to allow them to settle evenly on the bottom of the well
    # Arrange them in the following order:

    #  -----------------------------------------------------------------------------------------
    # | marker |         Batch 1          |         Batch 2          |         Batch 3          |
    # | --------------------------------------------------------------------------------------- |
    # | marker | Before | After  | Sample | Before | After  | Sample | Before | After  | Sample |
    #  -----------------------------------------------------------------------------------------
    
    #  -----------------------------------------------------------------------------------------
    # | Col 1  | Col 2  | Col 3  | Col 4  | Col 5  | Col 6  | Col 7  | Col 8  | Col 9  | Col 10 |
    # | --------------------------------------------------------------------------------------- |
    # | marker | ID_B1  | ID_A1  | ID_I1  | ID_B2  | ID_A2  | ID_I2  | ID_B3  | ID_A3  | ID_I3  |
    #  -----------------------------------------------------------------------------------------
    grab_lids(tank_count)
    
    load_marker(op_in_gel)
    
    load_sample_into_wells(op_count,op_in_protein,op_in_before,op_in_after,op_in_gel,amount_protein)

    # Place the lid on the tank and make sure to align it with the color-coded plugs.
    place_lid_and_align_to_plugs(op_in_protein,op_in_before,op_in_after)
    
    set_a_30mins_timer
    
    remove_gel_cassette(op_in_gel)
    
    # Remove the gels from the gel cassette by gently separating the two plates of the gel cassette.
    gel_removal
    
    # Remove the gel by floating it off the plate, inverting the gel and plate under water.
    
    # Soak the gel in water and put the container on the shaker in the incubator.
    gel_wash(op_in_gel)
    
    # Replace water with clean water every 5mins for three times.
    second_wash(op_in_gel)
    
    final_wash_add_staining(op_in_gel)
    
    # Put the container on the shaker in incubator for 1hour.
    put_container_on_shaker_1hr(op_in_gel)
    
    # Replace the water with clean water
    remove_gel_stain(op_in_gel)
    
    background_wash(op_in_gel)
    
    operations.each do |op|
        op.input("SDS Gel").item.mark_as_deleted
        op.output("SDS Gel").child_item.move "on the shaker in a still incubator"
      end
   
    operations.store
    get_protocol_feedback

    return {}
  end
  
  def grab_lids(tank_count)
    show do
        title "Set up the power supply"
        check "In the gel room, obtain a power supply."
        check "Grab <b>#{tank_count}</b> lid(s). Attach the electrodes of a lid to the power supply. Make sure to align the color-coded plugs and jacks (black to black / red to red)."
        image "Actions/ProteinPurification/lid.jpg"
    end
  end
  
  def load_marker(op_in_gel)
    show do
        title "Add protein ladder to gel"
        check "Grab the protein ladder from a box label <b>protein purification</b> in -20°C freezer (B1-165)."
        check "Pipette 10 µl of the protein ladder to well position 1 (the leftmost well) of gel #{op_in_gel.to_sentence}."
    end
  end

  def load_sample_into_wells(op_count,op_in_protein,op_in_before,op_in_after,op_in_gel,amount_protein)
    show do 
      title "Load sample into gel"
      check "Grab a P200 pipettor and a box of gel loading tip."
      check "Load samples to the gel according to the table:"
      note "Load the sample slowly to allow it to settle evenly on the bottom of the well. Be careful not to puncture the bottom of gel wells."
      for i in 0...op_in_gel.size
        #note "Gel ID: #{op_in_gel[i]}"
        op_table = [["Gel ID","Well number","Sample ID","Loading amount"]]
        well_no = 2
        if (i+1)*3 > op_count
            length = op_count - (i*3)
        else
            length = 3
        end
        for j in 0...length
            # before
            row = []
            row << op_in_gel[i]
            row << well_no
            row << {content:op_in_before[i*3+j], check: true}
            row << "40 µl"
            op_table << row
            well_no = well_no + 1
            # after
            row = []
            row << op_in_gel[i]
            row << well_no
            row << {content:op_in_after[i*3+j], check: true}
            row << "40 µl"
            op_table << row
            well_no = well_no + 1
            # sample
            row = []
            row << op_in_gel[i]
            row << well_no
            row << {content:op_in_protein[i*3+j], check: true}
            row << "#{amount_protein[i*3+j]} µl"
            op_table << row
            well_no = well_no + 1
        end
        table op_table
      end
      check "If there is an empty well, load 10 µl of sample buffer into each empty well to avoid lateral diffusion."
      image "Actions/ProteinPurification/loading_tip.jpg"
    end
  end
    
  def place_lid_and_align_to_plugs(op_in_protein,op_in_before,op_in_after)
    show do 
      title "Start Electrophoresis"
      check "Attach the lid to tank. Make sure that the red electrode attaches to the red terminal of the power supply, and the black 
      electrode attaches to the neighboring black terminal (as shown in the picture)."
      check "Set the power supply to 80 V."
      check "Set a 20-min timer and hit the RUN button to start."
      check "while waiting, return the protein ladder, sample buffer, P200 pipettor and gel loading tip box."
      image "Actions/ProteinPurification/Start_Electrophoresis.jpg"
    end
  end
  
  def set_a_30mins_timer
      show do
        title "Electrophoresis"
        check "Set the power supply to 150 V."
        check "Set a 50-mins timer and hit the RUN button to start."
        check "Check on gels with a manager."
        bullet "If the protein ladder is fully separated and the green marker/blue line sits on the bottom of a gel, proceed to the next step."
        bullet "If not, the lab manager may have you set another timer."
      end
  end
  
  def remove_gel_cassette(op_in_gel)
    show do 
        title "Remove gel cassette"
        check "Turn off the power supply."
        check "Remove the tank lid."
        check "Pour off the running buffer into the sink."
        check "Grab <b>#{op_in_gel.size}</b> box(es) and label with ID: #{op_in_gel.to_sentence}."
        check "Fill the box half full of DI water."
        check "Open the arms of assembly and release the gel cassette."
        check "Place the gel cassette into the corresponding box (as shown in the picture)."
        image "Actions/ProteinPurification/empty_box.jpg"
    end
  end

  def gel_removal
    show do 
      title "Remove gel from the glass cassette"
      check "Grab a gel releaser."
      check "Gently separate the two plates of a gel cassette by using a gel releaser. (as shown in the picture 1)"
      check "Remove the short plate on the top."
      check "Cut away the top layer of gel by using a gel releaser. (as shown in the picture 2)"
      check "Inverting the glass plate under water. Floating the gel off the plate in the corresponding box."
      image "Actions/ProteinPurification/gel_releaser.jpg"
    end
  end

  def gel_wash(op_in_gel)
    show do 
      title "Wash gels"
      check "Place #{op_in_gel.to_sentence} on the shaker in a still incubator."
      check "Wash gels at 300 rpm for 5 minutes."
      timer initial: { hours: 0, minutes: 5, seconds: 0}
      check "While waiting, clean up the glass plates, running tanks and the gel releaser by rinsing with water."
      check "Remove the dot stickers on clamping frames and wash it by rinsing with water."
      check "Return the lids."
      image "Actions/ProteinPurification/shaker_in_incubator.jpg"
    end
  end
    
  def second_wash(op_in_gel)
    show do 
      title "Second wash"
      check "Retrieve #{op_in_gel.to_sentence} from shaker."
      check "Pour off water and fill the box half full of DI water."
      check "Return #{op_in_gel.to_sentence} to the shaker in a still incubator."
      check "Wash gels at 300 rpm for 5 minutes."
      timer initial: { hours: 0, minutes: 5, seconds: 0}
    end
  end
  
  def final_wash_add_staining(op_in_gel)
    show do
        title "Stain gels"
        check "Retrieve #{op_in_gel.to_sentence} from shaker."
        check "Pour off water."
        check "Grab a motorized pipet."
        check "Grab a gel staining buffer from 4°C refrigerator (R1-350)."
        check "Add 20 mL of gel staining buffer to each box."
        check "Return the gel staining buffer."
        image "Actions/ProteinPurification/gel_staining_buffer.jpg"
    end
  end

  def put_container_on_shaker_1hr(op_in_gel)
    show do 
      title "Stain gels"
      check "Place #{op_in_gel.to_sentence} on the shaker in a still incubator."
      check "Stain gel at 150 rpm for 1 hour."
      timer initial: { hours: 1, minutes: 0, seconds: 0}
    end
  end

  def remove_gel_stain(op_in_gel)
    show do 
      title "Remove gel staining buffer"
      check "Retrieve #{op_in_gel.to_sentence} from the shaker."
      check "Pour off gel staining buffer."
      check "Rinse gels with water twice."
      check "Fill the box half full of DI water."
    end
  end
  
  def background_wash(op_in_gel)
    show do
        title "Destain gels "
        check "Place #{op_in_gel.to_sentence} on a shaker in the still incubator."
        check "Wash gels at 300 rpm for 30 minutes."
        timer initial: { hours: 0, minutes: 30, seconds: 0}
    end
    
    show do
        title "Destain gels "
        check "Retrieve #{op_in_gel.to_sentence} from the shaker."
        check "Pour off water and fill the box half full of DI water."
        check "Place gel box on shaker in a still incubator."
        check "Wash gel overnight."
    end
  end
  
end
```
