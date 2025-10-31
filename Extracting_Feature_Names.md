This script is not strictly required; it simply automates what would otherwise be a manual and repetitive task. It
generates all branch and bus variable names for the IEEE 14-bus system using MATPOWER. Doing this by hand would be (1)
time-consuming, especially for larger networks, and (2) prone to errors when indexing or naming variables.

You can choose any other approach/language to achieve the same result. This just happens to be a quick method that works
well for me. You can even optimize it further; but all of this is happening at the data creation stage which is performed
offline and never interferes with the application process of any NN algorithm.

```mtlab
% Load the case data
mpc = loadcase('case14');

% Extract the branch data
branch_data = mpc.branch;

% Initialize a cell array to store the branch names
P_from_branch_names = {};
Q_from_branch_names = {};
I_from_branch_names = {};
Phi_from_branch_names = {};
I_to_branch_names = {};
Phi_to_branch_names = {};
To_Python = {};

% Loop through each branch and create the corresponding name
for i = 1:size(branch_data, 1)
    fbus = branch_data(i, 1);  % From bus
    tbus = branch_data(i, 2);  % To bus
    P_from_branch_names{i} = sprintf('''P_from_%d_to_%d''', fbus-1, tbus-1);
    Q_from_branch_names{i} = sprintf('''Q_from_%d_to_%d''', fbus-1, tbus-1);
    I_from_branch_names{i} = sprintf('''I_from_%d_to_%d''', fbus-1, tbus-1);
    Phi_from_branch_names{i} = sprintf('''Phi_from_%d_to_%d''', fbus-1, tbus-1);
    I_to_branch_names{i} = sprintf('''I_from_%d_to_%d''', tbus-1, fbus-1);
    Phi_to_branch_names{i} = sprintf('''Phi_from_%d_to_%d''', tbus-1, fbus-1);
    To_Python{i} = sprintf('[%d, %d]', fbus-1, tbus-1);
end

% Open a text file to write the branch names
fileID1 = fopen('P_from_branch_names_14bus.txt', 'w');
fileID2 = fopen('Q_from_branch_names_14bus.txt', 'w');
fileID3 = fopen('I_from_branch_names_14bus.txt', 'w');
fileID4 = fopen('Phi_from_branch_names_14bus.txt', 'w');
fileID5 = fopen('I_to_branch_names_14bus.txt', 'w');
fileID6 = fopen('Phi_to_branch_names_14bus.txt', 'w');
fileIDPy = fopen('To_Python_14.txt', 'w');

% Write each branch name to the file
for i = 1:length(P_from_branch_names)
    fprintf(fileID1, '%s,\n', P_from_branch_names{i});
end

for i = 1:length(Q_from_branch_names)
    fprintf(fileID2, '%s,\n', Q_from_branch_names{i});
end

for i = 1:length(I_from_branch_names)
    fprintf(fileID3, '%s,\n', I_from_branch_names{i});
end

for i = 1:length(Phi_from_branch_names)
    fprintf(fileID4, '%s,\n', Phi_from_branch_names{i});
end

for i = 1:length(I_to_branch_names)
    fprintf(fileID5, '%s,\n', I_to_branch_names{i});
end

for i = 1:length(Phi_to_branch_names)
    fprintf(fileID6, '%s,\n', Phi_to_branch_names{i});
end

for i = 1:length(To_Python)
    fprintf(fileIDPy, '%s,\n', To_Python{i});
end

% Close the file
fclose(fileID1);
fclose(fileID2);
fclose(fileID3);
fclose(fileID4);
fclose(fileID5);
fclose(fileID6);
fclose(fileIDPy);

%% Bus/Node Data
bus_data = mpc.bus;

P_bus_names = {};
Q_bus_names = {};
V_bus_names = {};
Theta_bus_names = {};

for i = 1:size(bus_data, 1)
    atbus = bus_data(i, 1);  % at bus

    P_bus_names{i} = sprintf('''P%d''', atbus-1);
    Q_bus_names{i} = sprintf('''Q%d''', atbus-1);
    V_bus_names{i} = sprintf('''V%d''', atbus-1);
    Theta_bus_names{i} = sprintf('''Theta_%d''', atbus-1);
end

fileID7 = fopen('Real_PI_14bus.txt', 'w');
fileID8 = fopen('Reactive_PI_14bus.txt', 'w');
fileID9 = fopen('V_14bus.txt', 'w');
fileID10 = fopen('Theta_14bus.txt', 'w');

for i = 1:length(P_bus_names)
    fprintf(fileID7, '%s,\n', P_bus_names{i});
end

for i = 1:length(Q_bus_names)
    fprintf(fileID8, '%s,\n', Q_bus_names{i});
end

for i = 1:length(V_bus_names)
    fprintf(fileID9, '%s,\n', V_bus_names{i});
end

for i = 1:length(Theta_bus_names)
    fprintf(fileID10, '%s,\n', Theta_bus_names{i});
end

fclose(fileID7);
fclose(fileID8);
fclose(fileID9);
fclose(fileID10);
```
