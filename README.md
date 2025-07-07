import numpy as np
import random
from time import time

# Configurations
n_vars = 10_000_000         # Number of Boolean variables
n_clauses = 10_000_000      # Number of clauses (3-SAT)
angles_per_var = 3          # Number of spectral angles per variable

print("✨ Generating spectral angles...")
start_time = time()

# Step 1: Generate 3 spectral angles per variable ∈ [0, π]
theta_multi = np.random.uniform(0, np.pi, size=(n_vars, angles_per_var))

# Step 2: Compute the average angle for each variable
theta_avg = np.mean(theta_multi, axis=1)

# Step 3: Compute logical assignment using cos
theta_cos = np.cos(theta_avg)
A = (theta_cos > 0).astype(np.int8)  # Logical assignment: 1 if cos > 0

print(f"✅ Assigned {n_vars:,} variables in {time() - start_time:.2f} seconds")

# Step 4: Generate random 3-SAT clauses
def generate_sat_clauses(n_vars, n_clauses):
    clauses = []
    for _ in range(n_clauses):
        vars_ = random.sample(range(1, n_vars + 1), 3)
        clause = [v * random.choice([1, -1]) for v in vars_]
        clauses.append(clause)
    return clauses

print("✨ Generating SAT clauses...")
start_time = time()
clauses = generate_sat_clauses(n_vars, n_clauses)
print(f"✅ Clauses generated in {time() - start_time:.2f} seconds")

# Step 5: Evaluate how many clauses are satisfied
def evaluate_assignment(clauses, assignment):
    satisfied = 0
    for clause in clauses:
        if any((assignment[abs(lit)-1] == 1) if lit > 0 else (assignment[abs(lit)-1] == 0) for lit in clause):
            satisfied += 1
    return satisfied

print("✨ Evaluating assignment accuracy...")
start_time = time()
satisfied_count = evaluate_assignment(clauses, A)
accuracy = satisfied_count / n_clauses

print("\n✨✨✨ RESULTS ✨✨✨")
print(f"Satisfied clauses: {satisfied_count:,} out of {n_clauses:,}")
print(f"Accuracy: {accuracy:.2%}")
print(f"⏱ Evaluation time: {time() - start_time:.2f} seconds")
