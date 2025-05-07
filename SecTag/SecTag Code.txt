#====================================================================
import pandas as pd
def sectag_to_regex(header_file_path, seg_col, header_col):
  header_df = pd.read_csv(header_file_path)
  header_df = header_df.drop_duplicates()
  headers = header_df[header_col].tolist()
  header_patterns = [f'^{header}[\n:]' for header in headers]
  return header_patterns, header_df[seg_col].tolist()

#====================================================================
import re
def find_segs(note, header_patterns, seg_names):
  segs = {}

  # Find the section headers and their start positions
  for i, pattern in enumerate(header_patterns):
    for m in re.finditer(pattern, note.lower(), re.MULTILINE):
      seg_head = (note[m.span()[0]:m.span()[1]], m.span()[0])
      if seg_head not in segs:
        segs[seg_head] = []  # A seg head can have multiple general seg names

      segs[seg_head].append(seg_names[i]) 

  segs = [[k[0], segs[k], k[1]] for k in segs.keys()]
  segs = sorted(segs, key=lambda x: x[2])
  
  # Find the entir sections and their start and end positions
  for i in range(len(segs)):
    if i == len(segs)-1:
      segs[i].append(len(note))
    else:
      segs[i].append(segs[i+1][2])

  return segs