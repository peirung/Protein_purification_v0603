# Scan Gel

The objective of this protocol is to verify protein expression.

- Workflow:
  - SDS-PAGE Analysis > *Scan Gel*
- Steps:
  - [01] Place the gel on a light box and take a picture.
  - [02] verify the protein expression.
### Inputs


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
# Modified from "Cloning/Extract Gel Slice"

needs "Standard Libs/UploadHelper"
needs "Standard Libs/AssociationManagement"
needs "Standard Libs/Feedback"


class Protocol
  include Feedback
  include UploadHelper, AssociationManagement
    
  # upload stuff
  DIRNAME="<where are gel files on computer>"
  TRIES=3
  
  def main

    operations.retrieve(interactive: true)
    
    take_out_filter
    
    # get gel images
    gels = operations.map { |op| op.input("SDS Gel").collection}.uniq
    gels.each do |gel|
    
        grouped_ops=operations.select { |op| op.input("SDS Gel").collection == gel }
        image_name = "gel_#{gel.id}"
        
        # image gel
        image_gel(gel,image_name)
        
        # upload image
        ups = uploadData("#{DIRNAME}/#{image_name}", 1, TRIES) # 1 file per gel
        ups = [Upload.find(1)] if debug
        
        up=nil
        if(!(ups.nil?))
            up=ups[0]

            gel_item = Item.find(gel.id)

            # associate gel image to gel
            gel_item.associate image_name, "successfully imaged gel", up
          
            grouped_ops.each do |op| # associate to all operations connected to gel
                # description of where this op is in the gel, to be used as desc tag for image upload
                location_in_gel = "#{op.input("SDS Gel").sample.name} is in row #{op.input("SDS Gel").row + 1} and column #{op.input("SDS Gel").column + 1}"
            
                # associate image to op with a location description
                op.associate image_name, location_in_gel, up
            
                # associate image to plan, or append new location to description if association already exists
                existing_assoc = op.plan.get(image_name)
                if existing_assoc && op.plan.upload(image_name) == up
                    op.plan.modify(image_name, existing_assoc.to_s + "\n" + location_in_gel, up)
                else
                    op.plan.associate image_name, location_in_gel , up
                end
            end
        end
        
        # check lengths of fragments in gel
        verify_protein_size(gel,grouped_ops)
        
        clean_up gel, gels
    end
    
    put_filter_back
    
    get_protocol_feedback

  end
  
  def check_gel_clean
    show do
        title "Check on gels"
        check "Check on gels with a manager."
        bullet "If blue dye has been removed from the gel matrix background and protein bands can be distinguished (as an example shown in the picture below), proceed to the next step ."
        bullet "If the background is still in dark blue, fill the box half full of fresh DI water. Wash gels for another 1 hour."
        image "Actions/ProteinPurification/gel_staining.jpg"
    end
  end
  
  def  take_out_filter
    show do
        title "Set up the camera"
        check "Take off the filter on the camera."
        check "Spray the light box with ethanol and wipe it dry with a paper towel."
        check "Place the light box on the UV light box for taking a photo. (as shown in the picture)"
        image "Actions/ProteinPurification/light_box.jpg"
    end
  end

  
  def image_gel(gel,image_name)
    show do
        title "Image gel #{gel}"
        check "Put the gel #{gel} on the light box and take a picture (protein marker is on the right side)."
        check "Check to see if the picture matches the gel before uploading."
        check "Rename the picture you just took exactly as <b>#{image_name}</b>"
    end
  end
  
  def verify_protein_size(gel,grouped_ops)
    op_table = [["Gel ID","Well Number","Batch - Sample","Expected protein expression"]]
    well_no = 2
    batch_no = 1

    grouped_ops.each do |op|
        # before
        row = []
        row << op.input("SDS Gel").item.id
        row << well_no
        row << "#{batch_no} - 1"
        #row << "Before IPTG"
        row << {content:"Uninduced", style:{color:"#90D"}}
        op_table << row
        well_no = well_no + 1
        # after
        row = []
        row << op.input("SDS Gel").item.id
        row << well_no
        row << "#{batch_no} - 2"
        #row << "After IPTG"
        row << {content: "Induced", style:{color:"#90D"}}
        op_table << row
        well_no = well_no + 1
        # sample
        row = []
        row << op.input("SDS Gel").item.id
        row << well_no
        row << "#{batch_no} - 3"
        #row << "Protein Sample"
        row << {content: "Purified", style:{color:"#90D"}}
        op_table << row
        well_no = well_no + 1
        
        batch_no = batch_no + 1
    end

    show {
        title "Verify protein expression"
        bullet "Verify samples on gel #{gel}:"
        table grouped_ops.start_table
          .custom_column(heading: "Gel ID") { |op| op.input("SDS Gel").item.id }
          .custom_column(heading: "Batch Number", checkable: false) { |op| op.input("SDS Gel").column + 1 }
          .custom_column(heading: "Expected Protein Size") { |op| op.input("SDS Gel").sample.properties["Size"] }
          .get(:correct, type: 'text', heading: "Does the bands of one data set match the expected result? (y/n)", default: 'y')
          .custom_column(heading: "User") { |op| op.user.name }
        .end_table
        bullet "<b>To verify a protein expression, every three wells are viewed as a data set for a batch. In the following table, 1-1, 1-2 and 1-3 are considered as one data set to verify a protein expression. The same condition applies to 2-1, 2-2 and 2-3 and so on.</b>"
        note"Expected protein expression of each well:"
        table op_table
        bullet "Example: expected protein expresssion for three kinds of proteins"
        image "Actions/ProteinPurification/expected_protein_expression.jpg"
    }
  end
  
  def clean_up gel, gels
      show {
        title "Discard gel"
        check "Dispose of the gel #{gel} into waste container."
      }
    end
    
    
 def put_filter_back
    show do
        title "Clean up"
        check "Spray the surface of the light box with ethanol and wipe it dry with a paper towel. Place it on the shelf."
        check "Reassemble the filter to the camera."
        check "Remove the label from the gel box."
        check "Clean up the gel box by rinsing with water. Return it to the gel station."
    end
  end

end
```
