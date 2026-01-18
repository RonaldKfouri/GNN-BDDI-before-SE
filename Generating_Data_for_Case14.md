This code uses the load datasets created in the previous notebook to generate the data that will be used for training and testing the GNN.  
At each time step, the active (P) and reactive (Q) demands at every load bus are updated from the CSV files, then the MATPOWER solver `runpf` computes the steady-state operating point, and the results that converge are stored in CSV files.  

We start by initializing MATPOWER and setting the solver options. We use the Newton-Raphson method with 100 iterations and suppress the verbose console output. This step is important if you are working on a virtual machine or high-performance computing environment so the memory wouldn't be full (of text that you would never read, especially for larger grids).  

```matlab
startup
clear;
clc;
define_constants;

% Load the MATPOWER case for the 14-bus system
mpc = loadcase('case14');
mpopt = mpoption('pf.nr.max_it', 100, 'verbose', 0, 'out.all', 0);
```
Then, the data are loaded and matrices are pre-allocated because MATLAB doesn't like to change the size of the matrix at every iteration. Each row in the CSV files represents one timestep (e.g., one minute), and each column corresponds to a specific load bus.  

We also determine the total number of timesteps, buses, and branches in the case.  

```matlab
% Load the modified power demands (P and Q) for each bus
LCData_P = readmatrix('~/Your_Path/P_modified_14bus.csv'); % Active power data
LCData_Q = readmatrix('~/Your_Path/Q_modified_14bus.csv'); % Reactive power data

% Initialize arrays to store results
[num_rows, ~] = size(LCData_P); % Number of timesteps
num_buses = size(mpc.bus, 1); % Number of buses
num_branches = size(mpc.branch, 1); % Number of branches

% Initialize result storage matrices
voltage_magnitudes = zeros(num_rows, num_buses);
voltage_angles = zeros(num_rows, num_buses);
real_p_injections = zeros(num_rows, num_buses);
reactive_p_injections = zeros(num_rows, num_buses);
real_power_from = zeros(num_rows, num_branches);
real_power_to = zeros(num_rows, num_branches);
reactive_power_from = zeros(num_rows, num_branches);
reactive_power_to = zeros(num_rows, num_branches);
branch_current_magnitudes_from = zeros(num_rows, num_branches);
branch_current_angles_from = zeros(num_rows, num_branches);
branch_current_magnitudes_to = zeros(num_rows, num_branches);
branch_current_angles_to = zeros(num_rows, num_branches);
```
Before running the power flow simulations, we must determine which buses in the MATPOWER case represent PQ load buses.  

These indices (`load_bus_indices`) will be used to update the corresponding entries in the bus matrix at each timestep with the values from the synthetic load datasets.  
```matlab
% Get the bus indices with loads
bus_types = mpc.bus(:, 2);
Pd = mpc.bus(:, PD); % Active power demand column
Qd = mpc.bus(:, QD); % Reactive power demand column
load_bus_indices = find(bus_types == 1 & Pd > 0); % Buses that have loads
```

At each timestep, we overwrite the MATPOWER bus demands at all PQ load buses using the corresponding row from the synthetic datasets, then run an AC power flow. When the solver succeeds, we save the values of interest.  

```matlab
% Loop through each time step and solve the power flow
for t = 1:num_rows
    % Update the load for each bus based on the CSV files
    for i = 1:length(load_bus_indices)
        bus_num = load_bus_indices(i);  % Bus number with load
        mpc.bus(bus_num, PD) = LCData_P(t, i) * 100;  % Update active load (MW)
        mpc.bus(bus_num, QD) = LCData_Q(t, i) * 100;  % Update reactive load (MVAR)
    end

    % Solve the power flow
    results = runpf(mpc, mpopt);
    
    if results.success == 1
        % Save bus-related results
        voltage_magnitudes(t, :) = results.bus(:, VM);
        voltage_angles(t, :) = results.bus(:, VA);
        for i = 1:num_buses
            Pg = sum(results.gen(results.gen(:, GEN_BUS) == i, PG));  % Real power generation
            Qg = sum(results.gen(results.gen(:, GEN_BUS) == i, QG));  % Reactive power generation
            Pd = results.bus(i, PD);  % Real power demand
            Qd = results.bus(i, QD);  % Reactive power demand
            real_p_injections(t, i) = Pg - Pd;
            reactive_p_injections(t, i) = Qg - Qd;
        end
        % Save branch-related results
        real_power_from(t, :) = results.branch(:, PF);
        real_power_to(t, :) = results.branch(:, PT);
        reactive_power_from(t, :) = results.branch(:, QF);
        reactive_power_to(t, :) = results.branch(:, QT);
        
        % Calculate current magnitudes and angles for 'from' and 'to' buses
        V_from = results.bus(results.branch(:, F_BUS), VM) .* exp(1j * deg2rad(results.bus(results.branch(:, F_BUS), VA)));
        V_to = results.bus(results.branch(:, T_BUS), VM) .* exp(1j * deg2rad(results.bus(results.branch(:, T_BUS), VA)));
        S_from = results.branch(:, PF) + 1j * results.branch(:, QF);
        S_to = results.branch(:, PT) + 1j * results.branch(:, QT);
        I_from = conj(S_from ./ V_from);
        I_to = conj(S_to ./ V_to);
        
        % Store current magnitudes and angles
        branch_current_magnitudes_from(t, :) = abs(I_from);
        branch_current_angles_from(t, :) = angle(I_from) * 180 / pi;
        branch_current_magnitudes_to(t, :) = abs(I_to);
        branch_current_angles_to(t, :) = angle(I_to) * 180 / pi;
    end
end
```

Finally, they are saved in CSV files.  
Note: we may not be using all the files at later stages but it is good to store them all so we wouldn't run iterations again.
```matlab
% Save results to CSV files
writematrix(voltage_magnitudes, '~/Your_Path/Voltage_Magnitudes_case14.csv');
writematrix(voltage_angles, '~/Your_Path/Voltage_Angles_case14.csv');
writematrix(real_p_injections, '~/Your_Path/Real_P_Injections_case14.csv');
writematrix(reactive_p_injections, '~/Your_Path/Reactive_P_Injections_case14.csv');
writematrix(real_power_from, '~/Your_Path/Real_Power_From_case14.csv');
writematrix(real_power_to, '~/Your_Path/Real_Power_To_case14.csv');
writematrix(reactive_power_from, '~/Your_Path/Reactive_Power_From_case14.csv');
writematrix(reactive_power_to, '~/Your_Path/Reactive_Power_To_case14.csv');
writematrix(branch_current_magnitudes_from, '~/Your_Path/Branch_Current_Magnitudes_From_case14.csv');
writematrix(branch_current_angles_from, '~/Your_Path/Branch_Current_Angles_From_case14.csv');
writematrix(branch_current_magnitudes_to, '~/Your_Path/Branch_Current_Magnitudes_To_case14.csv');
writematrix(branch_current_angles_to, '~/Your_Path/Branch_Current_Angles_To_case14.csv');

```
