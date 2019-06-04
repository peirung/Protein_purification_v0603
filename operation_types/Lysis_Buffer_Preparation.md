# Lysis Buffer Preparation

The objective of this protocol is to prepare lysis buffer for His-tagged protein purification workflow.

- Steps:
  - [01] Gather the chemical reagents.
  - [02] Prepare the buffer according to the receipe.
  - [03] Measure and adjust the pH value to 8.0
  - [04] Keep the buffer in 4°C refrigerator.




### Outputs


- **Lysis buffer** [L]  
  - <a href='#' onclick='easy_select("Sample Types", "Assay Buffer")'>Assay Buffer</a> / <a href='#' onclick='easy_select("Containers", "500 mL Liquid")'>500 mL Liquid</a>

### Precondition <a href='#' id='precondition'>[hide]</a>
```ruby
def precondition(_op)
  true
end
```

### Protocol Code <a href='#' id='protocol'>[hide]</a>
```ruby
#Pei 2019
#Recipe: 1X PBS, 1% Trition X-100, pH 7.4 EDTA-Free protease inhibitor(freshly add in the protocol of "Lyse Cell")


class Protocol

  def main

    operations.make
    
    show do
            title "Gather the Following Items:"
            check "a 600 mL beaker and label it with lysis buffer."
            check "10X PBS (pH 7.4)"
            check "Trition X-100"
    end
    
    show do
            title "Add following chemical reagents to a 600 mL beaker."
            check "300 mL of sterile DI water"
            note "**If there is no sterile water, use DI water and then filter the buffer by a top-bottle filter in the final step of this protocol.**"
            check "Using a serological pipette, add 50mL of 10X PBS to the beaker."
            check "Using a serological pipette, add 5 mL of Triton X-100 to the beaker."
            check "Fill the beaker with sterile DI water up to the 500-mL mark."
            check "Put a stir bar in the beaker and stir the buffer for 2 minutes."
    end
    
 
    show do
            title "Label the bottle"
            check "Grab a 500 mL bottle and label it with <b>Lysis Buffer, your initials, the date</b>"
            check "Pour the prepared buffer to bottle."
            note "If you are using non-sterile DI water, filter the buffer by a top-bottle filter in this step."
            check "Place the Lysis buffer in 4°C refrigerator(BO.110)."
    end
    
    show do
            title "Clean up"
            check "Place the used beaker in the dishwashing area."
    end
    
    
    operations.store interactive: false
    
    return {}
    
  end

end
```
