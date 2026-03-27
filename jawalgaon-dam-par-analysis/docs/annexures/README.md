# Annexures

EAP standard tables for submission to the Central Water Commission and State Dam Safety Organisation.

## Contents

| File | Description | Format |
|------|-------------|--------|
| `Annexure_3A_Piping.csv` | Settlement flood data — piping scenario | CSV |
| `Annexure_3A_Overtopping.csv` | Settlement flood data — overtopping scenario | CSV |
| `Annexure_3B_Piping_PAR.csv` | PAR summary by category — piping | CSV |
| `Annexure_3B_Overtopping_PAR.csv` | PAR summary by category — overtopping | CSV |

## CWC EAP Format Compliance

Annexure 3A/3B columns follow the format specified in:
> Central Water Commission (2018). *Guidelines for Preparation of Emergency Action Plan for Large Dams.* Chapter 5, Tables 5.3 and 5.4.

Column mapping between CWC template and this tool's output is documented in [DATA_DICTIONARY.md](../../DATA_DICTIONARY.md).

## How to Generate

Run the notebook through Cell 19. Files will be saved to `/content/PAR_Outputs/` and included in the final ZIP archive (Cell 22).
