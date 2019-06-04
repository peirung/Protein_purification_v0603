# SDS-PAGE Gel Casting

The objective of this protocol is to cast a gel for SDS-PAGE analysis.

- Workflow:
  - *SDS-PAGE Gel Casting* > SDS-PAGE Analysis
- Steps:
  - [01] Grab a short plate, spacer plate, casting frame, comb, clamping frame, gel running tank and casting stand with gray foam gasket.
  - [02] Slide two glass plates into the casting frame and engage the presssure cams to secure the glasses in the casting frame.
  - [03] Assemble the casting frame on the casting stand.
  - [04] Load water to the glass cassette sandwich and Check it with leakage.
  - [05] Pour off water and prepare 12% resolving gel solution.
  - [06] Load resolving gel solution into the glass cassette and overlay isopropanol.
  - [07] Wait for 1 hour and pour off isopropanol.
  - [08] Prepare 10% stacking gel solution and Load it to the glass cassette.
  - [09] Insert a comb and wait for 30 minutes.
  - [10] Remove the comb and release glass cassette from the casting frame.
  - [11] Slide the glass cassette into clamping frame and lock it into place.
  - [12] Place the clamping frame assembly into the gel running tank.
  - [13] Prepare running buffer and fill the gel running tank with running buffer.




### Outputs


- **SDS Gel** [G]  Part of collection
  - NO SAMPLE TYPE / <a href='#' onclick='easy_select("Containers", "12% SDS Gel in Tank")'>12% SDS Gel in Tank</a>

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
    op_count = operations.size

    # 1 batch           -> 3 inputs: before, after, iptg
    # 1 gel             -> max. 3 batches [marker, batch1, batch2, batch3]
    # 1 clamping frame  -> max. 2 gels
    # 1 tank            -> max. 2 frames
    
    gel_count = (op_count/3).floor
    remainder = op_count % 3
    if remainder != 0
        gel_count = gel_count + 1
    end
    
    before = operations.size
    
    operations.make
    
    op_out_gel = []
    operations.output_collections["SDS Gel"].each do |op_collection|
        op_out_gel << op_collection.id
    end
    
    clamping_frame_count = (op_count/6).floor
    remainder = op_count % 6
    if remainder != 0
        clamping_frame_count = clamping_frame_count + 1
    end
    
    running_buffer=0
    water=0
    for i in 1..clamping_frame_count
        if i%2 == 0
            running_buffer = running_buffer + 20
            water = 10*running_buffer - 20
        else
            running_buffer = running_buffer + 80
            water = 10*running_buffer - 80
        end
    end
    
    tank_count = (op_count/12).floor
    remainder = op_count % 12
    if remainder != 0
        tank_count = tank_count + 1
    end
    
    # Place the casting frame at upright coner with the pressure cams in the open position and facing to a flat surface.
    place_casting_frame(gel_count,clamping_frame_count)
        
    # Slide the two glass plates into the casting frame, keeping the short plate facing to front of the frame.
    place_short_plate_slide
        
    # Assemble the casting frame on the casting stand.
    place_assemble_frame_into_stand(clamping_frame_count)
        
    # Load water into glass cassette sandwich for testing leaking.
    test_leaking
    
    # - Add 0.01g APS powder to 100uL water
    prepare_aps_solution(gel_count)
    
    
    # Prepare 12% resolving gel
    prepare_resolving_gel(gel_count)
    
    # Add 2mL isopropanol on the top of the gel buffer in the gel cassette sandwich.
    wait_for_1h
        
    # Prepare one 10% stacking gel
    prepare_stacking_gel(gel_count)
     
    set_timer_30mins
    
    place_gel_into_clamping(clamping_frame_count)
    
    place_frames_into_tank(tank_count,clamping_frame_count)
    
    running_buffer_preparation(running_buffer,water)   
    
    pour_running_buffer
    
    label_gel_id(op_out_gel)
    
    clean_up
    
    operations.store
    get_protocol_feedback

    return {}
  end
  
  def place_casting_frame(gel_count,clamping_frame_count)
    show do
       title "Glass cassette preparation"
       check "Grab the following items:"
       note "#{gel_count} short plate"
       note "#{gel_count} spacer plate"
       note "#{gel_count} comb"
       note "#{gel_count} casting frame"
       #bullet "#{clamping_frame_count} casting stand"
       note "#{gel_count} gray foam gasket"
       check "Make sure the glass plates and comb are dry and clean. If not, spray ethanol on them and wipe them dry with Kimwipes."
       image "Actions/ProteinPurification/casting_tool.jpg"
    end
  end

  def place_short_plate_slide
    show do
       title "Assemble glass cassette sandwich"
       check "Grab a spacer plate and place a short plate on top of it (as shown in the picture 1)."
       check "Place a casting frame upright with the pressure cams in the open position."
       check "Slide glass plates into the casting frame (as shown in the picture 2)"
       check "Push plates vertically to the benchtop. Make sure both plates are flush at bottom."
       check "Engage pressure cams to secure the glass cassette in the casting frame (as shown in the picture 3)."
       check "Make sure the short plate is facing to front of the frame and the labeling on the spacer plate is up."
       check "To avoid leakage, put one finger at the bottom of plates and slide the finger to check both plates are flush at bottom."
       check "Repeat the steps above until all glass cassette sandwiches are assembled."
       image "Actions/ProteinPurification/casting_frame.jpg"
    end
  end
    
  def place_assemble_frame_into_stand(clamping_frame_count)
    show do
        title "Casting stand assembly" 
        check "Grab <b>#{clamping_frame_count}</b> casting stand(s) (as shown in the picture 1)."
        check "Place the gray foam gasket on the casting stand (as shown in the picture 2)."
        check "Clamp casting frame on casting stand (as shown in the picture 3)."
        image "Actions/ProteinPurification/casting_stand_new.jpg"
    end
  end
    
  def test_leaking
    show do
       title "Leak testing"
       check "Grab a motorized pipet."
       check "Pipet 10 mL of water into the glass cassette sandwich."
       check "Wait for 2 minutes. Check for leaks"
        timer initial: { hours: 0, minutes: 2, seconds: 0}
       bullet "If there is a leakage, wipe glass plates dry. Reassemble glass plates and make sure plates are flush at bottom."
       bullet "If there is no leakage, proceed to the next step."
       check "Remove water from glass cassette sandwich. Pour water to sink."
       check "Hold the casting stand upside down and remove remaining water in glass cassette with paper towel."
    end
  end
  
  def prepare_aps_solution(gel_count)
    show do
       title "Prepare 10% Ammonium persulfate solution"
       check "Grab a 1.5 mL tube and label with APS"
       check "Weigh #{gel_count*0.02} g of APS powder on a weighing paper and add the powder into the 1.5 mL tube."
       check "Pipette #{gel_count*200} µl water into the tube."
       check "Vortex until the powder is fully dissolved."
       image "Actions/ProteinPurification/APS.jpg"
    end
  end
  
  def prepare_resolving_gel(gel_count)
    op_table = [[" Order "," Reagent "," Volume "],
    ["1",{content:"Water", check: true},"#{gel_count*3.4} mL"],
    ["2",{content:"Acrylamide", check: true},"#{gel_count*4} mL"],
    ["3",{content:"Resolving gel buffer", check: true},"#{gel_count*2.5} mL"],
    ["4",{content:"10% SDS buffer", check: true},"#{gel_count*100} µl"],
    ["5",{content:"APS", check: true},"#{gel_count*100} µl"],
    ["6",{content:"TEMED", check: true},"#{gel_count*10} µl"],
    ["","Total volume","#{gel_count*10} mL"]
    ]

    show do
        title "Gel solution preparation"
        note  "Gather the following items"
        check "DI water"
        check "Resolving gel buffer"
        check "TEMED"
        check "Acrylamide (in 4°C refrigerator)"
        check "10% SDS buffer"
        check "APS solution (just fresh prepared)"
        check "a 50 mL falcon tube"
        check "Isopropanol"
        check "motorized pipet and a 10 mL serological pipette."
    end
    
    show do
        title "Prepare 12% resolving gel solution"
        note "Prepare a gel solution in the Falcon tube according to the table. Order is important."
        table op_table
        image "Actions/ProteinPurification/resolving_buffer.jpg"
    end
    
    show do
        title "Resolving gel casting"
        warning "Perform the following steps as quickly as possible. Gel solution turns solid in a short time."
        check "Pipette up and down three times by using a motorized pipet."
        check "Load 7.5 mL of gel solution to each gel cassette sandwich. Avoid air bubbles."
        check "Gently pipette 2 mL of isopropanol into glass cassette from left to right. Avoid to disturb the gel solution."
    end
  end

  def wait_for_1h
    show do
       title "Resolving gel polymerization"
       check "Set a 1-hr timer."
       timer initial: { hours: 1, minutes: 0, seconds: 0}
       check "While waiting, return the resolving gel buffer and isopropanol. Keep other reagents for later use."
       check "When the timer is up, pour off isopropanol into the sink. Remove remaining isopropanol with paper towel."
    end
  end         
  
  def prepare_stacking_gel(gel_count)
    op_table = [[" Order "," Reagent "," Volume "],
    ["1",{content:"Water", check: true},"#{gel_count*2.05} mL"],
    ["2",{content:"Acrylamide", check: true},"#{gel_count*1.65} mL"],
    ["3",{content:"Stacking gel buffer", check: true},"#{gel_count*1.25} mL"],
    ["4",{content:"10% SDS buffer", check: true},"#{gel_count*50} µl"],
    ["5",{content:"APS", check: true},"#{gel_count*25} µl"],
    ["6",{content:"TEMED", check: true},"#{gel_count*5} µl"],
    ["","Total volume","#{gel_count*5} mL"]
    ]
    
    show do
        title "Prepare stacking gel solution"
        check "Grab a 50 mL falcon tube, motorized pipet and a 10 mL serological pipette."
        check "Grab stacking gel buffer."
        note "Prepare a gel solution in the Falcon tube according to the table. Order is important."
        table op_table
    end
    
    show do
    title "Stacking gel casting"
        warning "Perform the following steps as quickly as possible. Gel solution turns solid in a short time."
        check "Pipette up and down three times by using a motorized pipet."
        check "Gently load 2.5 mL of gel solution to each gel cassette sandwich. Avoid air bubbles."
        check "Insert a comb into the gel cassette (as shown in the picture)."
        image "Actions/ProteinPurification/stacking_buffer_and_comb.jpg"
    end
  end

  def set_timer_30mins
    show do
        title "Stacking gel polymerization"
        bullet "Set a 40-mins timer." 
        timer initial: { hours: 0, minutes: 40, seconds: 0}
        check "While waiting, return all the reagents. Discard the APS solution."
    end
  end
  
  def place_gel_into_clamping(clamping_frame_count)
    show do
        title "Electrode assembly"
        check "Gently remove the comb from the gel cassette."
        check "Grab <b>#{clamping_frame_count}</b> clamping frame(s)."
        check "Set the clamping frame to the green arms open position. Make sure the red marking is on your left (as shown in the picture 1)."
        check "Release the glass cassette from the casting frame."
        check "Place a gel sandwich into the front side of the clamping frame with the short plate facing inward (as shown in picture 2)."
        note "**If an odd number of gels is being run, make sure to use a buffer dam.**"
        check "Make sure the glass cassettes are sitting on the bottom notches of the clamping frame. Lock gel cassettes by sliding the green arms of the clamping frame over the gels (as shown in picture 3)."
        image "Actions/ProteinPurification/clamping_frame.jpg"
    end
  end

  def place_frames_into_tank(tank_count,clamping_frame_count)
    show do
        title "Gel tank preparation"
        check "Grab <b>#{tank_count}</b> gel running tank(s)."
        check "Make sure the red marking on the top inside edge of tank is on your left (as shown in the picture 1)."
        check "Place the electrode assembly into the gel running tank. Make sure the red electrode is matching with the red marking on the gel running tank (as shown in the picture 2)."
        image "Actions/ProteinPurification/running_tank.jpg"
    end  
  end
  
  def running_buffer_preparation(running_buffer,water)
    show do
        title "Running buffer preparation"
        check "Grab a beaker and cylinder."
        check "Add #{water} mL of DI water to the beaker."
        check "Add #{running_buffer} mL of premixed 10x Tris/glycine/SDS running buffer to the beaker."
        check "Stir for 2 minutes."
        timer initial: { hours: 0, minutes: 2, seconds: 0}
        image "Actions/ProteinPurification/running_buffer.jpg"
    end
  end
  
  def pour_running_buffer
    show do
        title "Fill tank with Running buffer"
        check "Place the gel running tank near to the power supply in the gel room."
        check "Fill <b>up chamber</b> with running buffer to the brim (as shown in the picture 1)."
        check "Wait for 2 minutes. Check for leaks." 
        timer initial: { hours: 0, minutes: 2, seconds: 0}
        bullet "If there is a leakage, pour running buffer back to the beaker. Open the green arms of clamping frame and then reassemble glass plates on the clamping frame."
        bullet "If not, proceed to the next step."
        check "Fill the <b>lower chamber</b> with running buffer to the indicated level. (2-Gels or 4-Gels line markings on the running gel tank, as shown in the picture 2)"
        image "Actions/ProteinPurification/leaking_check.jpg"
    end
  end
  
  def label_gel_id(op_out_gel)
    show do 
        title "Label gel ID"
        check "Write ID: #{op_out_gel.to_sentence} on dot stickers."
        check "Affix the label on clamping frame. (as shown in the picture)"
        image "Actions/ProteinPurification/dot_label.jpg"
    end
  end
  
  def clean_up
    show do
        title "Clean up"
        check "Clean up combs, casting frames, casting stands and gray foam gaskets by rinsing with water."
        check "Discard all used Falcon tubes and 1.5 mL tubes."
        check "Return the motorized pipet."
    end
  end

end
```
