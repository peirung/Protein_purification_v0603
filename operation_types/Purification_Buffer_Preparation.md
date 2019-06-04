# Purification Buffer Preparation

The objective of this protocol is to prepare purification buffer for His-tagged protein purification workflow.

- Steps:
  - [01] Gather the chemical reagents.
  - [02] Prepare the buffer according to the receipe.
  - [03] Measure and adjust the pH value to 8.0
  - [04] Keep the buffer in 4°C refrigerator.




### Outputs


- **Purification Buffer** [P]  
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
#Recipe: 50mM sodium phosphate buffer, 300mM NaCl (molar mass: 58.44 g/mol), 0.01% Triton X-100, pH 7.7
#Sodium phosphate dibasic, 7-Hydrate (Na2HPO4.7H2O) FW 268.07
#Sodium phosphat monobasic, monohydrate (NaH2PO4.H2O) FW 137.99

class Protocol

  def main

    operations.make
    
    show do
            title "Gather the Following Items:"
            check "a 600 mL beaker"
            check "10X PBS (pH 7.4)"
            check "Sodium phosphate dibasic, 7-Hydrate (Na2HPO4.7H2O) FW 268.07 "
            check "Sodium phosphate monobasic, monohydrate (NaH2PO4.H2O) FW 137.99"
            check "Sodium chloride, NaCl"
            check "Trition X-100"
    end
    
    show do
            title "Add following chemical reagents to the 600 mL beaker"
            check "6.24 g of sodium phosphate dibasic, 7-Hydrate (Na2HPO4.7H2O)"
            check "0.24 g of sodium phospate monobasic, monohydrate (NaH2PO4.H2O)"
            check "1.46 g of NaCl"
            check "300 mL of sterile DI water"
            note "**If there is no sterile water, use DI water and then filter the buffer by a top-bottle filter in the final step of this protocol.**"
            check "Using a serological pipette, add 50mL of 10X PBS to the beaker"
            check "50 µl of Triton X-100"
            check "Put one stir bar into the beaker and stir the buffer until the reagents are fully dissolved."
    end
    
    show do
            title "pH value calibration"
            check "Grab pH meter from the drawer."
            check "Grab three 50 mL Falcon tubes."
            check "Label the tubes with pH 4, pH 7 and pH 10."
            check "Grab pH standard solutions: pH 4 (red), pH 7 (yellow), pH 10 (blue)."
            check "Distribute 45 mL of each pH standard solutions into three different tubes."
            check "Press <b>ON/OFF</b> button on the pH meter."
            check "Dip electrode about 2 cm into the pH 7 (yellow) standard buffer solution."
            check "Press the CAL button to enter calibration mode. The CAL indicator will be shown on the screen display."
            check "Allow about 30-40 seconds for the pH meter reading to stablize, then press the <b>HOLD/ENT</b> button to confirm the first calibration point."
            check "Rinse electrode by DI water and wipe it dry with KimWipes."
    end
    
    show do
            title "Second calibration point"
            check "Dip electrode about 2 cm into the pH 4 (red) standard buffer solution."
            check "Press the CAL button to enter calibration mode. The CAL indicator will be shown on the screen display."
            check "Allow about 30-40 seconds for the pH meter reading to stablize, then press the <b>HOLD/ENT</b> button to confirm the second calibration point."
            check "Rinse electrode by DI water and wipe it dry with KimWipes."
    end 
    
    show do
            title "Third calibration point"
            check "Dip electrode about 2 cm into the pH 10 (blue) standard buffer solution."
            check "Press the CAL button to enter calibration mode. The CAL indicator will be shown on the screen display."
            check "Allow about 30-40 seconds for the pH meter reading to stablize, then press the <b>HOLD/ENT</b> button to confirm the third calibration point."
            check "After the third point calibration, the meter will automatically return to the measurement mode."
            check "Rinse electrode by DI water and wipe it dry with KimWipes."
    end
    
    show do
            title "pH value adjustment"
            check "Dip electrode about 2 cm into the buffer in the 500 mL beaker."
            check "The pH value of the buffer should be around 7.7"
            check "Adjust pH value to 7.7 by HCl or 5M NaOH if pH value is out of expecting range."
            check "Press <b>ON/OFF</b> to turn off the meter."
            check "Rinse electrode by DI water and wipe it dry with KimWipes."
            check "Return the meter and three standard buffers."
    end
    show do
            title "Label the bottle"
            check "Fill the beaker with sterile DI water to the 500 mL-mark. Stir the buffer for 2 minutes."
            check "Grab a 500 mL bottle and label it with <b>Purification Buffer, your initials, the date</b>"
            check "Pour the prepared buffer to the bottle."
            note "If you are using non-sterile DI water, filter the buffer by a top-bottle filter in this step."
            check "Place the purification buffer in 4°C refrigerator(BO.110)."
    end
    
    show do
            title "Clean up"
            check "Return chemical reagents."
            check "Place the used beaker and stir bar to the dishwashing area."
    end
    
    
    operations.store interactive: false
    
    return {}
    
  end

end


```
