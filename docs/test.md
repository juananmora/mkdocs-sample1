---
hide:
  - navigation
---

# Overview
This is a very simple package containing two modules that implement vehicle and car classes. The documentation includes the API docstrings for the modules and a demo example in Jupyter notebook.

## System structure
```mermaid
%%{init: {'theme':'default'}}%%
flowchart LR
subgraph Vehicle
num1[(num_of_wheels)]
end
subgraph Car
num2[(num_of_wheels)]
brand[(brand)]
drive[drive]

end
Car --inherits--> Vehicle
```