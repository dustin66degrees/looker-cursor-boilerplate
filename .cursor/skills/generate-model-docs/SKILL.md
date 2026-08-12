---
name: generate-model-docs
description: Generate or refresh a LookML data dictionary CSV from the current project. Use when the user asks to generate model documentation, update the data dictionary, or export exposed fields and explores.
---

# Generate LookML Data Dictionary Documentation

## Instructions

When invoked, perform the following steps exactly:

1. **Create the Python Script:** Write the Python script below to a temporary file named `generate_csv.py` in the root of the workspace.
2. **Setup and Execute:** Run the following terminal command to set up a virtual environment, install dependencies, and execute the script:
   ```bash
   python3 -m venv .venv && source .venv/bin/activate && pip install lkml pandas && python3 generate_csv.py
   ```
3. **Clean Up:** Remove the temporary files and virtual environment:
   ```bash
   rm -rf .venv generate_csv.py
   ```
4. **Notify the User:** Confirm to the user that the documentation has been successfully updated at `.documents/output/model_fields.csv`.

---

### Python Script (`generate_csv.py`)

```python
import os
import lkml
import pandas as pd
import glob

def title_case(s):
    return ' '.join(word.capitalize() for word in s.split('_'))

def parse_lookml():
    views = {}
    explores = {}
    
    # Directories to search (current project; add imported/base project paths if needed)
    search_dirs = ['.']
    
    view_files = []
    for d in search_dirs:
        # Find all view files, sort to ensure 01_base comes before 02_standard
        view_files.extend(sorted(glob.glob(os.path.join(d, '**/*.view.lkml'), recursive=True)))
        
    for f in view_files:
        with open(f, 'r') as file:
            try:
                parsed = lkml.load(file)
                if 'views' in parsed:
                    for view in parsed['views']:
                        view_name = view.get('name')
                        if view_name:
                            # Strip '+' for refinements
                            if view_name.startswith('+'):
                                view_name = view_name[1:]
                                
                            # if view already exists (e.g. refinements), merge them
                            if view_name in views:
                                for k, v in view.items():
                                    if isinstance(v, list) and k in views[view_name]:
                                        views[view_name][k].extend(v)
                                    else:
                                        views[view_name][k] = v
                            else:
                                views[view_name] = view
            except Exception as e:
                print(f"Error parsing {f}: {e}")

    # Find all explore files (only in the current project, not in imported projects)
    explore_files = glob.glob('**/*.explore.lkml', recursive=True)
        
    for f in explore_files:
        with open(f, 'r') as file:
            try:
                parsed = lkml.load(file)
                if 'explores' in parsed:
                    for explore in parsed['explores']:
                        explore_name = explore.get('name')
                        if explore_name:
                            explores[explore_name] = explore
            except Exception as e:
                print(f"Error parsing {f}: {e}")

    return views, explores

def extract_fields_and_sets(views):
    fields_data = {}
    view_sets = {}
    
    for view_name, view in views.items():
        view_label = view.get('label', title_case(view_name))
        view_sets[view_name] = {}
        
        # Parse sets
        if 'sets' in view:
            for s in view['sets']:
                set_name = s.get('name')
                set_fields = s.get('fields', [])
                view_sets[view_name][set_name] = set_fields
                
        # Field types to look for
        field_types = ['dimensions', 'measures', 'dimension_groups', 'filters', 'parameters']
        
        for f_type in field_types:
            if f_type in view:
                for field in view[f_type]:
                    field_name = field.get('name')
                    field_key = f"{view_name}.{field_name}"
                    
                    if field_key not in fields_data:
                        fields_data[field_key] = {
                            'view_name': view_name,
                            'view_label': view_label,
                            'field_name': field_name,
                            'field_type': f_type,
                            'label': field.get('label', title_case(field_name)),
                            'hidden': field.get('hidden', 'no') == 'yes',
                            'description': field.get('description', '')
                        }
                    else:
                        # Update existing field with refinement properties
                        if 'label' in field:
                            fields_data[field_key]['label'] = field['label']
                        if 'hidden' in field:
                            fields_data[field_key]['hidden'] = field['hidden'] == 'yes'
                        if 'description' in field:
                            fields_data[field_key]['description'] = field['description']
                    
    return fields_data, view_sets

def resolve_explore_fields(explore, view_sets):
    included = set()
    excluded = set()
    
    if 'fields' not in explore:
        return None, set()
        
    for item in explore['fields']:
        if item.startswith('-'):
            excluded.add(item[1:])
        elif item.endswith('*'):
            prefix = item[:-1]
            if '.' in prefix:
                v_name, s_name = prefix.split('.', 1)
                if v_name in view_sets and s_name in view_sets[v_name]:
                    for f in view_sets[v_name][s_name]:
                        if '.' not in f:
                            included.add(f"{v_name}.{f}")
                        else:
                            included.add(f)
                else:
                    included.add(item)
            else:
                included.add(item)
        else:
            included.add(item)
            
    return included, excluded

def determine_exposure(fields_data, view_sets, explores):
    field_exposure = {k: set() for k in fields_data.keys()}
    field_aliases = {k: set() for k in fields_data.keys()}
    
    for exp_name, exp in explores.items():
        base_alias = exp.get('name', exp_name)
        base_underlying = exp.get('view_name', base_alias)
        base_view_label = exp.get('view_label')
        
        aliases = {base_alias: {'underlying': base_underlying, 'view_label': base_view_label}}
        
        if 'joins' in exp:
            for join in exp['joins']:
                j_alias = join.get('name')
                j_underlying = join.get('from', j_alias)
                j_view_label = join.get('view_label')
                aliases[j_alias] = {'underlying': j_underlying, 'view_label': j_view_label}
                
        included, excluded = resolve_explore_fields(exp, view_sets)
        
        for field_key, f_data in fields_data.items():
            v_name = f_data['view_name']
            f_name = f_data['field_name']
            
            if f_data['hidden']:
                continue
                
            # Generate possible names for dimension groups (time and duration)
            possible_names = [f_name]
            possible_keys = [field_key]
            
            if f_data['field_type'] == 'dimension_groups':
                tfs = ['raw', 'time', 'date', 'week', 'month', 'quarter', 'year', 'day_of_week', 'hour_of_day', 'month_name', 'day_of_month', 'day_of_year', 'time_of_day', 'minute', 'hour', 'day']
                for tf in tfs:
                    possible_names.append(f"{f_name}_{tf}")
                    possible_keys.append(f"{field_key}_{tf}")
                    tf_plural = tf + 's' if not tf.endswith('s') else tf
                    possible_names.append(f"{tf_plural}_{f_name}")
                    possible_keys.append(f"{v_name}.{tf_plural}_{f_name}")
                
            # Find all aliases that use this underlying view
            matching_aliases = [alias for alias, info in aliases.items() if info['underlying'] == v_name]
            
            for alias in matching_aliases:
                is_exposed = False
                
                if included is None:
                    is_exposed = True
                else:
                    # Check explicit inclusion
                    if any(pk in included or f"{alias}.{pn}" in included or f"{v_name}.{pn}" in included for pk, pn in zip(possible_keys, possible_names)):
                        is_exposed = True
                    # Check alias.* or v_name.*
                    elif f"{alias}.*" in included or f"{v_name}.*" in included:
                        is_exposed = True
                    else:
                        # Check sets
                        for inc in included:
                            if inc.endswith('*') and '.' in inc:
                                inc_alias, inc_s = inc[:-1].split('.', 1)
                                # The include could use the alias OR the underlying view name
                                if inc_alias == alias or inc_alias == v_name:
                                    if inc_s in view_sets.get(v_name, {}):
                                        if any(pn in view_sets[v_name][inc_s] or pk in view_sets[v_name][inc_s] for pk, pn in zip(possible_keys, possible_names)):
                                            is_exposed = True
                                            break
                
                # Check exclusions
                if any(pk in excluded or f"{alias}.{pn}" in excluded or f"{v_name}.{pn}" in excluded for pk, pn in zip(possible_keys, possible_names)):
                    is_exposed = False
                elif f"{alias}.*" in excluded or f"{v_name}.*" in excluded:
                    is_exposed = False
                    
                if is_exposed:
                    field_exposure[field_key].add(exp_name)
                    
                    # Determine the effective view label
                    eff_label = aliases[alias]['view_label']
                    if not eff_label:
                        eff_label = f_data['view_label']
                    if not eff_label:
                        eff_label = title_case(alias)
                        
                    field_aliases[field_key].add(eff_label)
                
    results = []
    explore_names = sorted(list(explores.keys()))
    
    for field_key, f_data in fields_data.items():
        exposed_in = field_exposure[field_key]
        
        row = {
            'User-Facing Field Name': f_data['label'],
            'User-Facing View Name(s)': ", ".join(sorted(list(field_aliases[field_key]))),
            'View Name': f_data['view_name'],
            'Field Name': f_data['field_name'],
            'Description': f_data['description']
        }
        
        # Add a column for each explore with an "X" if exposed
        for exp_name in explore_names:
            row[exp_name] = "X" if exp_name in exposed_in else ""
            
        results.append(row)
        
    return pd.DataFrame(results)

if __name__ == "__main__":
    views, explores = parse_lookml()
    fields_data, view_sets = extract_fields_and_sets(views)
    df = determine_exposure(fields_data, view_sets, explores)
    
    df = df.sort_values(by=['View Name', 'Field Name'])
    
    output_dir = '.documents/output'
    os.makedirs(output_dir, exist_ok=True)
    
    output_file = os.path.join(output_dir, 'model_fields.csv')
    df.to_csv(output_file, index=False)
    print(f"Successfully generated {output_file} with {len(df)} fields.")
```
