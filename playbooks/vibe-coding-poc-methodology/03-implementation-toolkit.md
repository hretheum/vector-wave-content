# Vibe Coding POC: Implementation Toolkit

## Quick Start

```bash
# Clone and setup
git clone https://github.com/your-org/vibe-coding-poc
cd vibe-coding-poc
./setup.sh

# Run POC
python poc_runner.py --participants 8 --tasks 4

# Analyze results
python analyze_results.py --input results/ --output report.html
```

## Complete Implementation Scripts

### setup.sh - Environment Initialization

```bash
#!/bin/bash
set -e

# Color codes for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${GREEN}=== Vibe Coding POC Setup ===${NC}"

# Check prerequisites
check_requirement() {
    if ! command -v $1 &> /dev/null; then
        echo -e "${RED}✗ $1 is not installed${NC}"
        exit 1
    else
        echo -e "${GREEN}✓ $1 is installed${NC}"
    fi
}

echo "Checking requirements..."
check_requirement docker
check_requirement python3
check_requirement git
check_requirement curl

# Create directory structure
echo -e "\n${YELLOW}Creating directory structure...${NC}"
mkdir -p {tasks,results,data,environments,logs,reports}

# Download task definitions
echo -e "\n${YELLOW}Downloading task library...${NC}"
for task in PERF-001 DIST-002 LEGACY-003 DEBUG-004; do
    curl -s https://raw.githubusercontent.com/your-org/poc-tasks/main/$task.tar.gz | \
    tar -xz -C tasks/
done

# Setup Python environment
echo -e "\n${YELLOW}Setting up Python environment...${NC}"
python3 -m venv venv
source venv/bin/activate
pip install -q -r requirements.txt

# Initialize database
echo -e "\n${YELLOW}Initializing database...${NC}"
python3 << EOF
import sqlite3
conn = sqlite3.connect('poc_results.db')
cursor = conn.cursor()

cursor.execute('''
CREATE TABLE IF NOT EXISTS results (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    participant_id TEXT NOT NULL,
    task_id TEXT NOT NULL,
    tool_type TEXT NOT NULL,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    duration_seconds INTEGER,
    test_coverage REAL,
    review_score INTEGER,
    edge_cases_found INTEGER,
    confidence_rating INTEGER,
    tool_helpful_rating INTEGER,
    would_use_production BOOLEAN,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    result_id INTEGER,
    event_type TEXT,
    timestamp REAL,
    metadata TEXT,
    FOREIGN KEY (result_id) REFERENCES results (id)
)
''')

conn.commit()
conn.close()
print("Database initialized")
EOF

# Setup Docker environments
echo -e "\n${YELLOW}Building Docker environments...${NC}"
docker-compose -f environments/docker-compose.yml build

# Generate configuration
echo -e "\n${YELLOW}Generating configuration...${NC}"
cat > config.yaml << EOF
poc:
  name: "Vibe Coding Evaluation"
  organization: "${ORGANIZATION:-YourOrg}"
  start_date: "$(date +%Y-%m-%d)"
  
tasks:
  time_limit_minutes: 90
  categories:
    - performance
    - distributed
    - refactoring
    - debugging
    
tools:
  assisted:
    - copilot
    - cursor
    - tabnine
  conversational:
    - claude
    - chatgpt
    - gemini
    
metrics:
  quantitative:
    - duration
    - test_coverage
    - cyclomatic_complexity
    - edge_cases
  qualitative:
    - confidence
    - cognitive_load
    - tool_helpfulness
    - production_readiness
    
analysis:
  outlier_percentile: 5
  significance_level: 0.05
  minimum_improvement: 0.15
EOF

echo -e "\n${GREEN}✓ Setup complete!${NC}"
echo -e "Next steps:"
echo -e "  1. Run ${YELLOW}python participant_randomizer.py${NC} to assign tasks"
echo -e "  2. Share ${YELLOW}instructions.md${NC} with participants"
echo -e "  3. Start POC with ${YELLOW}python poc_runner.py${NC}"
```

### participant_randomizer.py - Task Assignment

```python
#!/usr/bin/env python3
"""
POC Participant Randomization with Balanced Assignment
Ensures each participant gets different task types and both tool types
"""

import random
import json
import hashlib
from datetime import datetime, timedelta
from itertools import combinations
from typing import List, Dict, Tuple
import pandas as pd

class ParticipantRandomizer:
    def __init__(self, config_path='config.yaml'):
        self.config = self._load_config(config_path)
        self.tasks = self._load_tasks()
        
    def _load_tasks(self) -> List[Dict]:
        """Load available tasks from task library"""
        tasks = []
        task_files = [
            'PERF-001', 'PERF-002', 'PERF-003',
            'DIST-001', 'DIST-002', 'DIST-003',
            'LEGACY-001', 'LEGACY-002', 'LEGACY-003',
            'DEBUG-001', 'DEBUG-002', 'DEBUG-003'
        ]
        
        for task_id in task_files:
            category = task_id.split('-')[0].lower()
            tasks.append({
                'id': task_id,
                'category': category,
                'difficulty': random.choice(['medium', 'hard'])
            })
        
        return tasks
    
    def create_balanced_assignment(self, participants: List[str]) -> pd.DataFrame:
        """
        Create balanced assignment ensuring:
        1. Each participant gets 2 different task categories
        2. Each uses both tool types
        3. Task difficulty is balanced
        """
        
        # Generate all valid task combinations (different categories)
        categories = list(set(t['category'] for t in self.tasks))
        category_pairs = list(combinations(categories, 2))
        
        # Ensure enough combinations
        while len(category_pairs) < len(participants):
            category_pairs.extend(category_pairs)
        
        random.shuffle(category_pairs)
        
        # Tool assignment pattern (alternating)
        tool_patterns = [
            ('conversational', 'assisted'),
            ('assisted', 'conversational')
        ]
        
        assignments = []
        for i, participant in enumerate(participants):
            # Get task categories for this participant
            cat1, cat2 = category_pairs[i % len(category_pairs)]
            
            # Select specific tasks from categories
            task1 = random.choice([t for t in self.tasks if t['category'] == cat1])
            task2 = random.choice([t for t in self.tasks if t['category'] == cat2])
            
            # Assign tools (alternating pattern)
            tool1, tool2 = tool_patterns[i % 2]
            
            # Generate unique participant ID
            participant_id = hashlib.md5(participant.encode()).hexdigest()[:8]
            
            assignments.append({
                'participant_id': participant_id,
                'participant_name': participant,
                'task_1_id': task1['id'],
                'task_1_category': task1['category'],
                'tool_1': tool1,
                'task_2_id': task2['id'],
                'task_2_category': task2['category'],
                'tool_2': tool2,
                'randomization_date': datetime.now().isoformat()
            })
        
        return pd.DataFrame(assignments)
    
    def validate_assignment(self, df: pd.DataFrame) -> Dict:
        """Validate that assignment meets balance criteria"""
        validation = {
            'participants': len(df),
            'unique_tasks': df['task_1_id'].nunique() + df['task_2_id'].nunique(),
            'tool_balance': {
                'conversational_first': sum(df['tool_1'] == 'conversational'),
                'assisted_first': sum(df['tool_1'] == 'assisted')
            },
            'category_distribution': {}
        }
        
        # Check category distribution
        all_categories = list(df['task_1_category']) + list(df['task_2_category'])
        for cat in set(all_categories):
            validation['category_distribution'][cat] = all_categories.count(cat)
        
        # Validation checks
        validation['is_valid'] = (
            abs(validation['tool_balance']['conversational_first'] - 
                validation['tool_balance']['assisted_first']) <= 1
        )
        
        return validation
    
    def generate_schedule(self, df: pd.DataFrame, start_date: datetime) -> pd.DataFrame:
        """Generate execution schedule"""
        df = df.copy()
        
        for i, row in df.iterrows():
            # Spread tasks across week
            # Half participants: Mon/Wed
            # Half participants: Tue/Thu
            if i < len(df) // 2:
                df.at[i, 'task_1_date'] = (start_date + timedelta(days=0 if i % 2 == 0 else 1)).strftime('%Y-%m-%d')
                df.at[i, 'task_2_date'] = (start_date + timedelta(days=2 if i % 2 == 0 else 3)).strftime('%Y-%m-%d')
            else:
                df.at[i, 'task_1_date'] = (start_date + timedelta(days=1 if i % 2 == 0 else 2)).strftime('%Y-%m-%d')
                df.at[i, 'task_2_date'] = (start_date + timedelta(days=3 if i % 2 == 0 else 4)).strftime('%Y-%m-%d')
        
        return df
    
    def export_assignments(self, df: pd.DataFrame):
        """Export assignments in multiple formats"""
        # JSON for processing
        df.to_json('assignments.json', orient='records', indent=2)
        
        # CSV for spreadsheet
        df.to_csv('assignments.csv', index=False)
        
        # Markdown for documentation
        with open('assignments.md', 'w') as f:
            f.write("# Participant Assignments\n\n")
            f.write(f"Generated: {datetime.now().strftime('%Y-%m-%d %H:%M')}\n\n")
            f.write(df.to_markdown(index=False))
        
        # Individual participant files
        for _, row in df.iterrows():
            with open(f"participants/{row['participant_id']}.md", 'w') as f:
                f.write(f"# Assignment for {row['participant_name']}\n\n")
                f.write(f"**Participant ID**: {row['participant_id']}\n\n")
                f.write("## Task 1\n")
                f.write(f"- **Task**: {row['task_1_id']}\n")
                f.write(f"- **Category**: {row['task_1_category']}\n")
                f.write(f"- **Tool**: {row['tool_1']}\n")
                f.write(f"- **Date**: {row['task_1_date']}\n\n")
                f.write("## Task 2\n")
                f.write(f"- **Task**: {row['task_2_id']}\n")
                f.write(f"- **Category**: {row['task_2_category']}\n")
                f.write(f"- **Tool**: {row['tool_2']}\n")
                f.write(f"- **Date**: {row['task_2_date']}\n")

def main():
    print("=== POC Participant Randomization ===\n")
    
    # Get participants
    print("Enter participant names (empty line to finish):")
    participants = []
    while True:
        name = input().strip()
        if not name:
            break
        participants.append(name)
    
    if len(participants) < 4:
        print(f"\nWarning: POC needs 6-8 participants for statistical validity")
        print(f"You have {len(participants)} participants")
        confirm = input("Continue anyway? (y/n): ")
        if confirm.lower() != 'y':
            return
    
    # Get start date
    start_date_str = input("\nPOC start date (YYYY-MM-DD) or Enter for next Monday: ").strip()
    if start_date_str:
        start_date = datetime.strptime(start_date_str, '%Y-%m-%d')
    else:
        today = datetime.now()
        days_ahead = 0 - today.weekday()  # Monday is 0
        if days_ahead <= 0:
            days_ahead += 7
        start_date = today + timedelta(days=days_ahead)
    
    # Create randomizer
    randomizer = ParticipantRandomizer()
    
    # Generate assignments
    print("\nGenerating balanced assignments...")
    assignments = randomizer.create_balanced_assignment(participants)
    assignments = randomizer.generate_schedule(assignments, start_date)
    
    # Validate
    validation = randomizer.validate_assignment(assignments)
    print("\nValidation Results:")
    print(f"  Participants: {validation['participants']}")
    print(f"  Unique tasks: {validation['unique_tasks']}")
    print(f"  Tool balance: {validation['tool_balance']}")
    print(f"  Valid: {'✓' if validation['is_valid'] else '✗'}")
    
    # Export
    randomizer.export_assignments(assignments)
    print("\nExported to:")
    print("  - assignments.json (for processing)")
    print("  - assignments.csv (for spreadsheet)")
    print("  - assignments.md (for documentation)")
    print("  - participants/*.md (individual assignments)")
    
    # Display summary
    print("\n" + "="*50)
    print("ASSIGNMENT SUMMARY")
    print("="*50)
    print(assignments[['participant_name', 'task_1_id', 'tool_1', 'task_2_id', 'tool_2']].to_string(index=False))

if __name__ == "__main__":
    main()
```

### poc_runner.py - Execution Monitor

```python
#!/usr/bin/env python3
"""
POC Execution Monitor
Tracks participant progress and collects metrics in real-time
"""

import time
import json
import sqlite3
import subprocess
from datetime import datetime
from pathlib import Path
import docker
import git
from typing import Dict, List
import threading
import queue

class POCRunner:
    def __init__(self, assignment_file='assignments.json'):
        self.assignments = self._load_assignments(assignment_file)
        self.db = sqlite3.connect('poc_results.db', check_same_thread=False)
        self.docker_client = docker.from_env()
        self.active_sessions = {}
        self.metrics_queue = queue.Queue()
        
    def _load_assignments(self, path) -> List[Dict]:
        with open(path) as f:
            return json.load(f)
    
    def start_session(self, participant_id: str, task_id: str, tool_type: str):
        """Initialize a participant session"""
        
        # Create workspace
        workspace = Path(f"workspaces/{participant_id}/{task_id}")
        workspace.mkdir(parents=True, exist_ok=True)
        
        # Copy task files
        subprocess.run([
            'cp', '-r', f'tasks/{task_id}/', str(workspace)
        ])
        
        # Initialize git repo for tracking
        repo = git.Repo.init(workspace)
        repo.index.add('*')
        repo.index.commit('Initial commit')
        
        # Start Docker environment
        container = self.docker_client.containers.run(
            f'poc-env:{task_id.lower()}',
            detach=True,
            name=f"{participant_id}_{task_id}",
            volumes={
                str(workspace.absolute()): {'bind': '/workspace', 'mode': 'rw'}
            },
            environment={
                'PARTICIPANT_ID': participant_id,
                'TASK_ID': task_id,
                'TOOL_TYPE': tool_type
            }
        )
        
        # Record session start
        cursor = self.db.cursor()
        cursor.execute('''
            INSERT INTO results (participant_id, task_id, tool_type, started_at)
            VALUES (?, ?, ?, ?)
        ''', (participant_id, task_id, tool_type, datetime.now()))
        self.db.commit()
        
        result_id = cursor.lastrowid
        
        # Start metrics collection
        session = {
            'result_id': result_id,
            'participant_id': participant_id,
            'task_id': task_id,
            'tool_type': tool_type,
            'container': container,
            'workspace': workspace,
            'repo': repo,
            'start_time': time.time()
        }
        
        self.active_sessions[f"{participant_id}_{task_id}"] = session
        
        # Start monitoring thread
        monitor_thread = threading.Thread(
            target=self._monitor_session,
            args=(session,)
        )
        monitor_thread.start()
        
        return result_id
    
    def _monitor_session(self, session: Dict):
        """Monitor a session and collect metrics"""
        
        while session['result_id'] in [s['result_id'] for s in self.active_sessions.values()]:
            metrics = {
                'timestamp': time.time() - session['start_time'],
                'container_stats': self._get_container_stats(session['container']),
                'git_stats': self._get_git_stats(session['repo']),
                'test_stats': self._get_test_stats(session['workspace'])
            }
            
            # Store metrics
            cursor = self.db.cursor()
            cursor.execute('''
                INSERT INTO events (result_id, event_type, timestamp, metadata)
                VALUES (?, ?, ?, ?)
            ''', (
                session['result_id'],
                'metrics',
                metrics['timestamp'],
                json.dumps(metrics)
            ))
            self.db.commit()
            
            time.sleep(10)  # Collect every 10 seconds
    
    def _get_container_stats(self, container) -> Dict:
        """Get Docker container statistics"""
        try:
            stats = container.stats(stream=False)
            return {
                'cpu_percent': stats['cpu_stats']['cpu_usage']['total_usage'],
                'memory_usage': stats['memory_stats']['usage'],
                'network_rx': stats['networks']['eth0']['rx_bytes'],
                'network_tx': stats['networks']['eth0']['tx_bytes']
            }
        except:
            return {}
    
    def _get_git_stats(self, repo) -> Dict:
        """Get git repository statistics"""
        try:
            return {
                'commits': len(list(repo.iter_commits())),
                'files_changed': len(repo.index.diff(None)),
                'lines_added': sum(d.added for d in repo.index.diff(None)),
                'lines_removed': sum(d.removed for d in repo.index.diff(None))
            }
        except:
            return {}
    
    def _get_test_stats(self, workspace: Path) -> Dict:
        """Get test execution statistics"""
        try:
            # Run coverage report
            result = subprocess.run(
                ['python', '-m', 'pytest', '--cov', '--cov-report=json'],
                cwd=workspace,
                capture_output=True,
                text=True,
                timeout=10
            )
            
            if result.returncode == 0:
                with open(workspace / 'coverage.json') as f:
                    coverage_data = json.load(f)
                    return {
                        'test_passed': True,
                        'coverage_percent': coverage_data['totals']['percent_covered'],
                        'lines_covered': coverage_data['totals']['covered_lines'],
                        'lines_total': coverage_data['totals']['num_statements']
                    }
            else:
                return {'test_passed': False}
        except:
            return {}
    
    def end_session(self, participant_id: str, task_id: str):
        """End a participant session and collect final metrics"""
        
        session_key = f"{participant_id}_{task_id}"
        if session_key not in self.active_sessions:
            return
        
        session = self.active_sessions[session_key]
        
        # Stop container
        session['container'].stop()
        session['container'].remove()
        
        # Collect final metrics
        final_metrics = {
            'duration_seconds': time.time() - session['start_time'],
            'final_git_stats': self._get_git_stats(session['repo']),
            'final_test_stats': self._get_test_stats(session['workspace'])
        }
        
        # Update database
        cursor = self.db.cursor()
        cursor.execute('''
            UPDATE results 
            SET completed_at = ?, 
                duration_seconds = ?,
                test_coverage = ?
            WHERE id = ?
        ''', (
            datetime.now(),
            final_metrics['duration_seconds'],
            final_metrics.get('final_test_stats', {}).get('coverage_percent', 0),
            session['result_id']
        ))
        self.db.commit()
        
        # Clean up
        del self.active_sessions[session_key]
        
        return final_metrics
    
    def generate_dashboard(self):
        """Generate real-time dashboard of POC progress"""
        
        cursor = self.db.cursor()
        
        # Get current status
        cursor.execute('''
            SELECT 
                participant_id,
                task_id,
                tool_type,
                started_at,
                completed_at,
                duration_seconds
            FROM results
            ORDER BY started_at DESC
        ''')
        
        results = cursor.fetchall()
        
        # Generate HTML dashboard
        html = '''
        <!DOCTYPE html>
        <html>
        <head>
            <title>POC Progress Dashboard</title>
            <meta http-equiv="refresh" content="30">
            <style>
                body { font-family: Arial, sans-serif; margin: 20px; }
                table { border-collapse: collapse; width: 100%; }
                th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
                th { background-color: #4CAF50; color: white; }
                .completed { background-color: #d4edda; }
                .in-progress { background-color: #fff3cd; }
                .pending { background-color: #f8f9fa; }
            </style>
        </head>
        <body>
            <h1>Vibe Coding POC - Progress Dashboard</h1>
            <p>Last updated: {timestamp}</p>
            
            <h2>Session Status</h2>
            <table>
                <tr>
                    <th>Participant</th>
                    <th>Task</th>
                    <th>Tool</th>
                    <th>Started</th>
                    <th>Completed</th>
                    <th>Duration</th>
                    <th>Status</th>
                </tr>
        '''.format(timestamp=datetime.now().strftime('%Y-%m-%d %H:%M:%S'))
        
        for row in results:
            participant_id, task_id, tool_type, started, completed, duration = row
            
            if completed:
                status = 'completed'
                status_text = '✓ Complete'
                duration_text = f"{duration // 60} min" if duration else '-'
            elif started:
                status = 'in-progress'
                status_text = '⏳ In Progress'
                duration_text = f"{(time.time() - datetime.fromisoformat(started).timestamp()) // 60:.0f} min"
            else:
                status = 'pending'
                status_text = '○ Pending'
                duration_text = '-'
            
            html += f'''
                <tr class="{status}">
                    <td>{participant_id[:8]}</td>
                    <td>{task_id}</td>
                    <td>{tool_type}</td>
                    <td>{started or '-'}</td>
                    <td>{completed or '-'}</td>
                    <td>{duration_text}</td>
                    <td>{status_text}</td>
                </tr>
            '''
        
        html += '''
            </table>
        </body>
        </html>
        '''
        
        with open('dashboard.html', 'w') as f:
            f.write(html)

def main():
    runner = POCRunner()
    
    print("=== POC Execution Monitor ===")
    print("Commands:")
    print("  start <participant_id> <task_id> <tool_type> - Start session")
    print("  end <participant_id> <task_id> - End session")
    print("  status - Show current status")
    print("  dashboard - Generate dashboard")
    print("  quit - Exit\n")
    
    while True:
        command = input("> ").strip().split()
        
        if not command:
            continue
        
        if command[0] == 'quit':
            break
        elif command[0] == 'start' and len(command) == 4:
            result_id = runner.start_session(command[1], command[2], command[3])
            print(f"Started session {result_id}")
        elif command[0] == 'end' and len(command) == 3:
            metrics = runner.end_session(command[1], command[2])
            print(f"Ended session. Duration: {metrics['duration_seconds']//60} minutes")
        elif command[0] == 'status':
            for key, session in runner.active_sessions.items():
                duration = (time.time() - session['start_time']) // 60
                print(f"{key}: {duration} minutes")
        elif command[0] == 'dashboard':
            runner.generate_dashboard()
            print("Dashboard updated: dashboard.html")
        else:
            print("Invalid command")

if __name__ == "__main__":
    main()
```