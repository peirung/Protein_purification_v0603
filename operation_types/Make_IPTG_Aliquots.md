# Make IPTG Aliquots

The objective of this protocol is to prepare IPTG aliquots for His-tagged protein purification workflow.

- Steps:
  - [01] Gather the chemical reagents.
  - [02] Prepare the buffer according to the receipe.
  - [03] Keep aliquots in -20°C freezer.




### Outputs


- **IPTG Aliquot** [I]  
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
#Recipe: 0.5M IPTG solution (final concentration in the 450 mL of cell culture is 0.5mM), molar mass of IPTG(Isopropyl beta-D-1-thiogalactopyranoside): 238.30 g/mol

class Protocol

  def main

    operations.make
    
    show do
            title "Gather the Following Items:"
            check "<b>18</b> 1.5 mL tubes and label them with <b>IPTG</b>"
            check "a bottle of IPTG powder (1g / per bottle)"
    end
    
    show do
            title "Add water to the bottle"
            check "Add 8.3 mL of sterile DI water"
            check "Mix the solution until IPTG powder is fully dissolved."
    end

    show do
            title "Make aliquots"
            check "Distribute 450 µl of the IPTG-buffer into the 1.5 mL tubes."
            check "Place the aliquots in a box labeled <b>protein purification</b> in -20°C freezer (B1-165)."
    end
    
    show do
            title "Clean up"
            check "Discard the empty bottle."
    end
    
    
    operations.store interactive: false
    
    return {}
    
  end

end
```
