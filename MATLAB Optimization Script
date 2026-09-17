% MAIN LIFECYCLE OPTIMIZATION SCRIPT (PHASE 3)
clear; clc;

dT_range = 2:1:40;
annual_opex = zeros(length(dT_range), 1);
capital_capex = zeros(length(dT_range), 1);
total_lifecycle_cost = zeros(length(dT_range), 1);

% Cost Parameters
steam_cost_per_kWh = 0.05; 
water_cost_per_kWh = 0.01;
operating_hours = 8400;
project_life_years = 10;

% Approximating Overall Heat Transfer Coefficient U (kW/m²°C)
U = 0.5; 

for k = 1:length(dT_range)
    current_dT = dT_range(k);
    
    % Executing Rigorous Thermodynamic Engine and Area Targeter
    [Q_hot_utility_min, Q_cold_utility_min, estimated_area] = pinch_engine_rigorous(current_dT, U);
    
    % Calculating Operating Cost per Year
    annual_opex(k) = (Q_hot_utility_min * steam_cost_per_kWh * operating_hours) + ...
                     (Q_cold_utility_min * water_cost_per_kWh * operating_hours);
                 
    % Equipment Costing Structure formula: Cost = C_fixed + C_area * (Area)^0.8
    if estimated_area > 0
        capital_capex(k) = 15000 + 800 * (estimated_area)^0.8;
    else
        capital_capex(k) = 0;
    end
    
    % 10-Year Simple Total Lifecycle Cost Model
    total_lifecycle_cost(k) = capital_capex(k) + (annual_opex(k) * project_life_years);
end

% Locate the exact financial optimum
[min_cost, optimum_index] = min(total_lifecycle_cost);
optimum_dT = dT_range(optimum_index);

% Displaying Optimal Design Point 
fprintf('Optimal Delta T Min            : %d °C\n', optimum_dT);
fprintf('Minimum 10-Year Lifecycle Cost : $%.2f USD\n', min_cost);
fprintf('==================================================\n');

% Plot Optimization Trade-off Curves
figure;
plot(dT_range, annual_opex * project_life_years / 1e3, 'r--', 'LineWidth', 1.5); hold on;
plot(dT_range, capital_capex / 1e3, 'b-.', 'LineWidth', 1.5);
plot(dT_range, total_lifecycle_cost / 1e3, 'g-', 'LineWidth', 2);
plot(optimum_dT, min_cost / 1e3, 'kx', 'MarkerSize', 12, 'LineWidth', 2);

xlabel('\DeltaT_{min} Approach Temperature (°C)');
ylabel('10-Year Cumulative Cost ($ x10^3 USD)');
title('Process Optimization: Capital vs. Operating Expenditure Trade-off');
legend('Operating Expense (10-Yr OpEx)', 'Capital Expense (CapEx)', 'Total Cost of Ownership', 'Thermodynamic Optimum');
grid on;

% Display the exact financial optimization figures
fprintf('\n==================================================\n');
fprintf('         FINAL PROCESS OPTIMUM RESULTS            \n');
fprintf('==================================================\n');
fprintf('Optimal Approach Temperature   : %d °C\n', optimum_dT);
% Back-calculate the true optimal utility flowrates at the 18°C
opt_Q_hot = annual_opex(optimum_index) / (steam_cost_per_kWh * operating_hours);
opt_Q_cold = 0; % System hits a threshold problem at 18°C, so cooling water drops to 0

fprintf('Required Hot Utility (Steam)   : %.1f kW\n', opt_Q_hot);
fprintf('Required Cold Utility (Water)  : %.1f kW\n', opt_Q_cold);

fprintf('One-time Network CapEx Investment: $%.2f USD\n', capital_capex(optimum_index));
fprintf('Annual Plant OpEx Expenses     : $%.2f USD/yr\n', annual_opex(optimum_index));
fprintf('--------------------------------------------------\n');
fprintf('FINAL TOTAL LIFECYCLE COST : $%.2f USD\n', min_cost);
fprintf('==================================================\n');


%% PHASE 2: CORE PINCH THERMODYNAMIC ENGINE 
function [Q_hot, Q_cold, total_area] = pinch_engine_rigorous(dT_min, U_global)
    % 1. Stream Definition Matrix [Type, Ts, Tt, Cp]
    streams = [
        1, 250, 60, 30;  % Hot Stream H1
        1, 200, 120, 80; % Hot Stream H2
        2, 40, 180, 45;  % Cold Stream C1
        2, 110, 230, 60  % Cold Stream C2
    ]; 
    num_streams = size(streams, 1); 

    % 2. Temperature Shifting Step 
    shifted_T = zeros(num_streams, 2);
    for i = 1:num_streams
        if streams(i,1) == 1
            shifted_T(i,1) = streams(i,2) - (dT_min/2);
            shifted_T(i,2) = streams(i,3) - (dT_min/2);
        else 
            shifted_T(i,1) = streams(i,2) + (dT_min/2);
            shifted_T(i,2) = streams(i,3) + (dT_min/2);
        end 
    end 

    % 3. Extracting and Sorting Unique Temperature Intervals
    all_T = unique(shifted_T(:)); 
    intervals = sort(all_T, 'descend');
    num_intervals = length(intervals) - 1; 

    % 4. Calculating Net Heat Balance (dH) within intervals
    dH = zeros(num_intervals, 1);
    CP_hot_int = zeros(num_intervals, 1);
    CP_cold_int = zeros(num_intervals, 1);
    
    for j = 1:num_intervals
        T_high = intervals(j);
        T_low = intervals(j+1);
        
        for i = 1:num_streams 
            s_high = max(shifted_T(i,1), shifted_T(i,2));
            s_low = min(shifted_T(i,1), shifted_T(i,2));

            if (s_high >= T_high) && (s_low <= T_low)
                if streams(i,1) == 1 
                    CP_hot_int(j) = CP_hot_int(j) + streams(i,4);
                else 
                    CP_cold_int(j) = CP_cold_int(j) + streams(i,4);
                end 
            end 
        end
        dH(j) = (CP_hot_int(j) - CP_cold_int(j)) * (T_high - T_low); 
    end

    % 5. Executing Heat Cascade Accumulation 
    base_cascade = zeros(num_intervals + 1, 1);
    for j = 1:num_intervals
        base_cascade(j+1) = base_cascade(j) + dH(j);
    end
    
    % Checking for negative heat flows 
    min_heat = min(base_cascade);
    if min_heat < 0 
        heat_injection_top = -min_heat; 
    else 
        heat_injection_top = 0;
    end
    
    % Generating final feasible heat cascade profile 
    feasible_cascade = base_cascade + heat_injection_top;

    % Identifying Utility Targets
    Q_hot = feasible_cascade(1);
    Q_cold = feasible_cascade(end); 

    % 6. Rigorous Network Area Sizing (Townsend & Linnhoff Bath Formula)
    total_area = 0;
    
    for j = 1:num_intervals
        % Return to actual, unshifted terminal temperatures for this interval block
        Th_high = intervals(j) + (dT_min / 2);
        Th_low  = intervals(j+1) + (dT_min / 2);
        Tc_high = intervals(j) - (dT_min / 2);
        Tc_low  = intervals(j+1) - (dT_min / 2);
        
        % Total heat exchanged vertically within this interval block
        Q_exchanged = (CP_hot_int(j) * (Th_high - Th_low)) + (CP_cold_int(j) * (Tc_high - Tc_low));
        
        if Q_exchanged > 0 && CP_hot_int(j) > 0 && CP_cold_int(j) > 0
            % Terminal temperature driving forces
            dT1 = Th_high - Tc_high;
            dT2 = Th_low - Tc_low;
            
            % Safe LMTD calculation guarding against equal driving forces
            if abs(dT1 - dT2) < 1e-5
                LMTD = dT1;
            else
                LMTD = (dT1 - dT2) / log(dT1 / dT2);
            end
            
            % Add interval surface area to network target total
            total_area = total_area + (Q_exchanged / (U_global * LMTD));
        end
    end
    
    % Add external utility heat exchanger surface areas 
    if Q_hot > 0
        total_area = total_area + (Q_hot / (U_global * 50)); % Assuming ~50°C utility driving force
    end
    if Q_cold > 0
        total_area = total_area + (Q_cold / (U_global * 20)); % Assuming ~20°C utility driving force
    end
end
