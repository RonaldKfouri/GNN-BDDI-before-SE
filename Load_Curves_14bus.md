The following code generates random active (P) and reactive (Q) load samples for 3 buses of a 14-bus system. The samples are non-sqeuential (i.i.d.) and based on 5 representative 24-hour load profiles with random scaling.

* It requires MATPOWER to be installed.

Let's start with initializing the workspace and definine the hourly load modifiers.  
Note: these are taken from paper [13] cited in the manuscript.   
[13] A. Akagic and I. Džafić, “Enhancing smart grid resilience with deep learning 
anomaly detection prior to state estimation,” Engineering Applications of Artificial 
Intelligence, vol. 127, p. 107368, Jan. 2024.  

```matlab
startup % optional
modifiers = [0.45, 0.21, 0.15, 0.12, 0.15, 0.21, 0.31, 0.55, 0.82, 0.91, 0.99, 0.98, 0.95, 0.93, 0.82, 0.78, 0.82, 0.95, 0.65, 0.42, 0.29, 0.45, 0.61, 0.51;
             0.59, 0.66, 0.39, 0.19, 0.05, 0.04, 0.22, 0.24, 0.43, 0.68, 0.82, 0.84, 0.75, 0.63, 0.75, 0.67, 0.65, 0.81, 0.89, 0.58, 0.38, 0.27, 0.63, 0.82;
             0.78, 0.75, 0.55, 0.39, 0.16, 0.19, 0.01, 0.14, 0.19, 0.43, 0.69, 0.85, 0.69, 0.42, 0.45, 0.43, 0.44, 0.45, 0.65, 0.52, 0.34, 0.25, 0.39, 0.82;
             0.59, 0.43, 0.20, 0.28, 0.32, 0.28, 0.35, 0.45, 0.65, 0.70, 0.78, 0.88, 0.98, 0.95, 0.89, 0.79, 0.86, 0.90, 0.75, 0.68, 0.50, 0.37, 0.35, 0.45;
             0.68, 0.72, 0.65, 0.55, 0.60, 0.45, 0.50, 0.39, 0.55, 0.62, 0.69, 0.81, 0.83, 0.90, 0.95, 0.97, 0.92, 0.90, 0.84, 0.82, 0.60, 0.45, 0.55, 0.60];

```
The loads are extracted using the following:

```matlab
mpc = loadcase('case14');

% Extract active and reactive power loads (Pd and Qd)
bus_types = mpc.bus(:, 2); % Bus types (1 = PQ load, 2 = PV, 3 = slack)
Pd = mpc.bus(:, 3);  % Active power demand
Qd = mpc.bus(:, 4);  % Reactive power demand

% Find the indices of buses with non-zero demand
load_bus_indices = find(bus_types == 1 & Pd > 0);
num_loads = length(load_bus_indices);

% Extract the loads for buses with non-zero demand
orig_loads_P0 = (Pd(load_bus_indices) / mpc.baseMVA)';  % Normalize by base MVA
orig_loads_Q0 = (Qd(load_bus_indices) / mpc.baseMVA)';  % Normalize by base MVA
```
 
The simulation duration is set and the containers for results are defined next. The dataset covers 2 years at 1-minute resolution.

```matlab
% Determine the total number of iterations
total_minutes = 2*365*24*60;

% Preallocate cell arrays to collect results from each parallel iteration
P_modified_cell = cell(2*365, 1);
Q_modified_cell = cell(2*365, 1);

% Set the random generator to default to ensure reproducibility
rng('default');

distr = @(x) 0.8 + (1.2-0.8).*rand(x, 1);
distr2 = @(x) 0.9 + (1.1-0.9).*rand(x, 1);
```
This section generates the randomized load samples day by day. For each day:
 
* We create temporary matrices `temp_P` and `temp_Q` of size 1440 × num_loads (1-minute resolution).
* For every hour, minute, and load bus, we:
1. Pick a random profile row from `modifiers`  
2. Use the current `hour` column from that profile  
3. Multiply by two independent noise scalars `distr` and `distr2`  
4. Scale the nominal per-unit loads `orig_loads_P0`, `orig_loads_Q0`  

This produces i.i.d. samples with realistic hourly weighting, which we later concatenate from the per-day cell arrays.  
(If Parallel Computing Toolbox is available, you can replace `for` with `parfor` to parallelize over days).

```matlab
for day = 1:2*365 % Optional: parallelize the loop over days using parfor
    % Create temporary matrices for this worker
    temp_P = zeros(24*60, num_loads);
    temp_Q = zeros(24*60, num_loads);
    
    for hour = 1:24
        for minute = 1:60
            for i_load = 1:num_loads
                modifier_id = randi(size(modifiers, 1)); 
                mod_load_i = hour;
                mod_1 = modifiers(modifier_id, mod_load_i) * distr(1) * distr2(1);
                mod_2 = modifiers(modifier_id, mod_load_i) * distr(1) * distr2(1);
                temp_P((hour-1)*60 + minute, i_load) = orig_loads_P0(i_load) * mod_1;
                temp_Q((hour-1)*60 + minute, i_load) = orig_loads_Q0(i_load) * mod_2;
            end
        end
    end
    
    % Store the temporary matrices in the cell arrays
    P_modified_cell{day} = temp_P;
    Q_modified_cell{day} = temp_Q;
end
```

Finally, both matrices are saved in CSV files.
```matlab
% Concatenate the results
P_modified = vertcat(P_modified_cell{:});
Q_modified = vertcat(Q_modified_cell{:});

% Write the results to CSV files
writematrix(P_modified, '~/Your_Path/P_modified_14bus.csv');
writematrix(Q_modified, '~/Your_Path/Q_modified_14bus.csv');
```