[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on002578-blue)](https://doi.org/10.82901/nemar.on002578)

Data for this selective attention task was collected in 2004
at the Swartz Center for Computational Neuroscience at UCSD.
These datasets are part of a larger corpus of 32-channel data
collected a few years prior. The experiment is identical
although the number of channel is larger (256), the electrode
positions are scanned and the anatomical MRI is provided
(allowing for precise source localization). See publication
for more details.

Raw data manipulation before export:
- Fuse all BDF BIOSEMI files and reference to electrode 135 (see loadallbdf_2020.m)
- Fuse with presentation file information (see loadallbdf_2020.m)
- Remove spurious events of type 'condition' and '201' (see clean_events.m)
- Add HED tags (see addHEDTags.m)
- Convert MRI to NIFTI format (MRIcron) and reorient (MRIcrogl) (see convert_nifti.m)
