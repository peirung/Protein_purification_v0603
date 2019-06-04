# Make Imidazole Aliquots

The objective of this protocol is to prepare imidazole aliquots for His-tagged protein purification workflow.

- Steps:
  - [01] Gather the chemical reagents.
  - [02] Prepare the buffer according to the receipe.
  - [03] Measure and adjust the pH value to 8.0
  - [04] Keep aliquots in -20°C freezer.




### Outputs


- **Imidazole Aliquot** [I]  
  - <a href='#' onclick='easy_select("Sample Types", "Assay Buffer")'>Assay Buffer</a> / NO CONTAINER

### Precondition <a href='#' id='precondition'>[hide]</a>
```ruby
def precondition(_op)
  true
end
```

### Protocol Code <a href='#' id='protocol'>[hide]</a>
```ruby
#Pei, 2019
#Recipe: 5M imidazole solution,pH 8.0 molar mass of imidazole: 68.077 g/mol

class Protocol

  def main

    operations.make
    
    show do
            title "Gather the Following Items:"
            check "<b>17</b> 2 mL tubes and label them with <b>imidazile</b>"
            check "imidazole"
            check "a 50 mL beaker"
    end
    
    show do
            title "Add following reagents to the beaker"
            check "10.2 g of imidazole."
            check "15 mL of sterile DI water"
            check "Put a stir bar in the beaker and stir the buffer on the stir plate until imidazole is fully dissolved."
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
            check "Use 5M NaOH or HCl solution to adjust the pH value to pH 8.0"
            note "Continuely stir the buffer on a stir plate."
            note "Wait for 1 minutes to let the reading stablize. Make sure the pH value is 8.0"
            check "Press <b>ON/OFF</b> to turn off the meter."
            check "Rinse electrode by DI water and wipe it dry with KimWipes."
            check "Return the meter and three standard buffers."
    end

    show do
            title "Make aliquots"
            check "Fill the beaker with sterile DI water to the 30 mL mark. Stir the buffer for 2 minutes."
            check "Aliquot 1800 µl of the buffer into each 2 mL tubes."
            check "Place the aliquots in a box labeled <b>protein purification</b> in -20°C freezer (B1-165)."
    end
    
    show do
            title "Clean up"
            check "Discard the used beaker and stir bar."
    end
    
    
    operations.store interactive: false
    
    return {}
    
  end

end
```
