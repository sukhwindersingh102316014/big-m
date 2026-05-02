clc
clear all

M = 1e6;

% Objective function (Minimize)
c = [2 8 0 0 M M];  

% Constraint matrix
A = [5 10 0 0 1 0;
    1 0 1 0 0 0;
    0 1 0 -1 0 1];

b = [150;20;14];

[m n] = size(A);

% Initial basis (Artificial + slack as needed)
bv_index = [5 3 6];   % A1, S1, A2

Y = [A b];

for s = 1:50

    cb = c(bv_index);
    Xb = Y(:,end);

    z = cb * Xb;

    % Zj - Cj for minimization
    zjcj = cb * Y(:,1:n) - c;

    Table = [zjcj z; Y]

    % Optimality condition (all >= 0)
    if all(zjcj <= 0)
        disp('Optimal solution achieved');
        Xb
        basic_variables = bv_index
        fprintf("Optimal objective function value = %f\n", z);
        break
    end

    % Entering variable (most negative)
    [a, EV] = max(zjcj);

    % Unbounded check
    if all(Y(:,EV) <= 0)
        disp("Unbounded solution")
        break
    end

    % Ratio test
    ratio = inf(m,1);
    for j = 1:m
        if Y(j,EV) > 0
            ratio(j) = Xb(j) / Y(j,EV);
        end
    end

    [k, LV] = min(ratio);

    % Update basis
    bv_index(LV) = EV;

    % Pivot
    pivot = Y(LV,EV);
    Y(LV,:) = Y(LV,:) / pivot;

    for i = 1:m
        if i ~= LV
            Y(i,:) = Y(i,:) - Y(i,EV) * Y(LV,:);
        end
    end
end

% Check artificial variables
if any(ismember(bv_index, [5 6]))
    disp('Artificial variables still in basis → infeasible solution');
else
    disp('Feasible optimal solution found');
end
