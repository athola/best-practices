# 7.4 Data Consistency and Transactions

## Overview

Data consistency and transactions are fundamental concerns in distributed systems. As systems grow and components become more distributed, maintaining data consistency becomes increasingly challenging.

## Consistency Philosophy

"Data consistency is not an absolute—it's a spectrum. The key is to choose the right level of consistency for your specific use case and understand the trade-offs involved."

### Consistency Principles

**Principle 1: Understand Your Consistency Requirements**
"Not all data needs to be perfectly consistent. Understand your business requirements and choose the appropriate consistency model."

**Principle 2: Embrace Eventual Consistency**
"In distributed systems, eventual consistency is often the most practical approach. Design your system to work with temporary inconsistencies."

**Principle 3: Design for Conflicts**
"Conflicts are inevitable in distributed systems. Design your conflict resolution strategies upfront rather than trying to prevent all conflicts."

**Principle 4: Monitor Consistency**
"Data consistency issues can be subtle. Monitor your system for consistency problems and set up alerts for anomalies."

## Consistency Models

### Strong Consistency

**Strong Consistency Pattern**
```python
# Recommended approach for strong consistency
class StrongConsistencyManager:
    def __init__(self, database, lock_manager):
        self.database = database
        self.lock_manager = lock_manager
    
    def update_data(self, key, value):
        """Update data with strong consistency guarantees"""
        # Acquire distributed lock
        lock = self.lock_manager.acquire_lock(key, timeout=30)
        try:
            # Read current value
            current_value = self.database.read(key)
            
            # Apply update
            new_value = self._apply_update(current_value, value)
            
            # Write new value
            self.database.write(key, new_value)
            
            # Ensure all replicas are synchronized
            self._wait_for_replication(key, new_value)
            
            return new_value
            
        finally:
            # Release lock
            self.lock_manager.release_lock(lock)
    
    def _wait_for_replication(self, key, value):
        """Wait for data to be replicated to all nodes"""
        timeout = 10
        start_time = time.time()
        
        while time.time() - start_time < timeout:
            if self._is_replicated(key, value):
                return
            time.sleep(0.1)
        
        raise ReplicationTimeoutError()
```

**When to Use Strong Consistency**
- Financial transactions
- Inventory management
- User authentication and authorization
- Critical business operations
- When immediate consistency is required

### Eventual Consistency

**Eventual Consistency Pattern**
```python
# Recommended approach for eventual consistency
class EventualConsistencyManager:
    def __init__(self, database, event_bus):
        self.database = database
        self.event_bus = event_bus
        self.conflict_resolver = ConflictResolver()
    
    def update_data(self, key, value):
        """Update data with eventual consistency"""
        # Generate unique version for this update
        version = self._generate_version()
        
        # Create update event
        event = UpdateEvent(
            key=key,
            value=value,
            version=version,
            timestamp=datetime.now()
        )
        
        # Store update locally
        self.database.store_update(event)
        
        # Publish update event
        self.event_bus.publish(event)
        
        return UpdateResult(success=True, version=version)
    
    def read_data(self, key):
        """Read data with conflict resolution"""
        # Get all versions of the data
        versions = self.database.get_versions(key)
        
        if not versions:
            return None
        
        # Resolve conflicts if multiple versions exist
        if len(versions) > 1:
            resolved_value = self.conflict_resolver.resolve(versions)
            # Store resolved value
            self.database.store_resolved(key, resolved_value)
            return resolved_value
        
        return versions[0].value
    
    def handle_update_event(self, event):
        """Handle update event from other nodes"""
        # Check for conflicts
        existing_versions = self.database.get_versions(event.key)
        
        if existing_versions:
            # Resolve conflict
            resolved_value = self.conflict_resolver.resolve(
                existing_versions + [event]
            )
            self.database.store_resolved(event.key, resolved_value)
        else:
            # No conflict, store directly
            self.database.store_update(event)
```

**Conflict Resolution Strategies**
```python
# Recommended approach for conflict resolution
class ConflictResolver:
    def resolve(self, versions):
        """Resolve conflicts between multiple versions"""
        # Sort by timestamp (last write wins)
        sorted_versions = sorted(versions, key=lambda v: v.timestamp)
        
        # Apply business-specific conflict resolution
        return self._apply_business_rules(sorted_versions)
    
    def _apply_business_rules(self, versions):
        """Apply business-specific conflict resolution rules"""
        # Example: For numeric values, use the maximum
        if all(isinstance(v.value, (int, float)) for v in versions):
            return max(v.value for v in versions)
        
        # Example: For lists, merge unique items
        if all(isinstance(v.value, list) for v in versions):
            merged = []
            seen = set()
            for version in reversed(versions):  # Reverse for priority
                for item in version.value:
                    if item not in seen:
                        merged.append(item)
                        seen.add(item)
            return list(reversed(merged))  # Restore original order
        
        # Default: last write wins
        return versions[-1].value
```

**When to Use Eventual Consistency**
- Social media feeds
- Content management systems
- Analytics and reporting
- High-availability requirements
- When performance is more important than immediate consistency

### Causal Consistency

**Causal Consistency Pattern**
```python
# Recommended approach for causal consistency
class CausalConsistencyManager:
    def __init__(self, database, vector_clock):
        self.database = database
        self.vector_clock = vector_clock
    
    def update_data(self, key, value, dependencies=None):
        """Update data with causal consistency"""
        # Update vector clock
        self.vector_clock.increment()
        
        # Create causal context
        causal_context = CausalContext(
            vector_clock=self.vector_clock.copy(),
            dependencies=dependencies or []
        )
        
        # Store update with causal context
        update = CausalUpdate(
            key=key,
            value=value,
            context=causal_context
        )
        
        self.database.store_update(update)
        
        return update
    
    def read_data(self, key):
        """Read data respecting causal dependencies"""
        # Get all updates for the key
        updates = self.database.get_updates(key)
        
        # Filter updates based on causal dependencies
        ready_updates = self._filter_causally_ready(updates)
        
        if not ready_updates:
            return None
        
        # Apply updates in causal order
        final_value = self._apply_causal_order(ready_updates)
        
        return final_value
    
    def _filter_causally_ready(self, updates):
        """Filter updates that have all dependencies satisfied"""
        ready_updates = []
        
        for update in updates:
            if self._are_dependencies_satisfied(update):
                ready_updates.append(update)
        
        return ready_updates
    
    def _are_dependencies_satisfied(self, update):
        """Check if all causal dependencies are satisfied"""
        for dep in update.context.dependencies:
            if not self._is_dependency_satisfied(dep):
                return False
        return True
```

## Transaction Management

### Distributed Transactions

**Two-Phase Commit Pattern**
```python
# Recommended approach for two-phase commit
class TwoPhaseCommitCoordinator:
    def __init__(self, participants):
        self.participants = participants
        self.transaction_log = TransactionLog()
    
    def execute_transaction(self, operations):
        """Execute distributed transaction using two-phase commit"""
        # Generate unique transaction ID
        transaction_id = self._generate_transaction_id()
        
        # Phase 1: Prepare
        prepare_results = self._prepare_phase(transaction_id, operations)
        
        # Check if all participants are ready
        if all(result.ready for result in prepare_results):
            # Phase 2: Commit
            self._commit_phase(transaction_id)
            return TransactionResult(success=True)
        else:
            # Abort transaction
            self._abort_phase(transaction_id)
            return TransactionResult(success=False, error="Transaction aborted")
    
    def _prepare_phase(self, transaction_id, operations):
        """Execute prepare phase"""
        prepare_results = []
        
        for participant, operation in zip(self.participants, operations):
            try:
                result = participant.prepare(transaction_id, operation)
                prepare_results.append(result)
            except Exception as e:
                # If any participant fails, abort all
                self._abort_participants(transaction_id)
                raise TransactionError(f"Prepare phase failed: {e}")
        
        return prepare_results
    
    def _commit_phase(self, transaction_id):
        """Execute commit phase"""
        for participant in self.participants:
            try:
                participant.commit(transaction_id)
            except Exception as e:
                # Log error but continue with other participants
                logger.error(f"Commit failed for participant: {e}")
    
    def _abort_phase(self, transaction_id):
        """Execute abort phase"""
        for participant in self.participants:
            try:
                participant.abort(transaction_id)
            except Exception as e:
                logger.error(f"Abort failed for participant: {e}")
```

### Saga Pattern

**Saga Pattern Implementation**
```python
# Recommended approach for saga pattern
class SagaOrchestrator:
    def __init__(self):
        self.steps = []
        self.compensations = []
        self.saga_log = SagaLog()
    
    def add_step(self, step, compensation):
        """Add step and its compensation"""
        self.steps.append(step)
        self.compensations.append(compensation)
    
    def execute_saga(self, context):
        """Execute saga with compensation"""
        saga_id = self._generate_saga_id()
        completed_steps = []
        
        try:
            # Execute each step
            for i, step in enumerate(self.steps):
                step_result = step.execute(context)
                completed_steps.append(i)
                
                # Log step completion
                self.saga_log.log_step_completion(saga_id, i, step_result)
            
            return SagaResult(success=True)
            
        except Exception as e:
            # Execute compensations in reverse order
            self._execute_compensations(saga_id, completed_steps, context)
            return SagaResult(success=False, error=str(e))
    
    def _execute_compensations(self, saga_id, completed_steps, context):
        """Execute compensations for completed steps"""
        for i in reversed(completed_steps):
            compensation = self.compensations[i]
            try:
                compensation.execute(context)
                self.saga_log.log_compensation_completion(saga_id, i)
            except Exception as e:
                logger.error(f"Compensation failed for step {i}: {e}")
                # Continue with other compensations
```

**Outbox Pattern**
```python
# Recommended approach for outbox pattern
class OutboxPattern:
    def __init__(self, database, message_bus):
        self.database = database
        self.message_bus = message_bus
        self.message_dispatcher = MessageDispatcher()
    
    def execute_transaction(self, operations, events):
        """Execute transaction with outbox pattern"""
        # Start transaction
        with self.database.transaction() as tx:
            try:
                # Execute business operations
                for operation in operations:
                    operation.execute(tx)
                
                # Store events in outbox
                for event in events:
                    self._store_outbox_message(tx, event)
                
                # Commit transaction
                tx.commit()
                
                # Dispatch events after commit
                self._dispatch_outbox_messages()
                
                return TransactionResult(success=True)
                
            except Exception as e:
                tx.rollback()
                raise TransactionError(f"Transaction failed: {e}")
    
    def _store_outbox_message(self, tx, event):
        """Store event in outbox table"""
        message = OutboxMessage(
            id=self._generate_message_id(),
            event_type=event.type,
            event_data=event.data,
            status='pending',
            created_at=datetime.now()
        )
        
        tx.insert('outbox', message)
    
    def _dispatch_outbox_messages(self):
        """Dispatch pending outbox messages"""
        pending_messages = self.database.query(
            "SELECT * FROM outbox WHERE status = 'pending'"
        )
        
        for message in pending_messages:
            try:
                self.message_bus.publish(
                    message.event_type,
                    message.event_data
                )
                
                # Mark as dispatched
                self.database.update(
                    'outbox',
                    {'status': 'dispatched'},
                    {'id': message.id}
                )
                
            except Exception as e:
                logger.error(f"Failed to dispatch message {message.id}: {e}")
                # Retry later
```

## Data Consistency Monitoring

**Consistency Monitoring**
```python
# Recommended approach for consistency monitoring
class ConsistencyMonitor:
    def __init__(self, data_sources, alert_manager):
        self.data_sources = data_sources
        self.alert_manager = alert_manager
        self.consistency_checks = []
    
    def add_consistency_check(self, check):
        """Add consistency check"""
        self.consistency_checks.append(check)
    
    def run_checks(self):
        """Run all consistency checks"""
        results = []
        
        for check in self.consistency_checks:
            try:
                result = check.execute(self.data_sources)
                results.append(result)
                
                if not result.is_consistent:
                    self.alert_manager.send_alert(
                        f"Consistency check failed: {check.name}",
                        result.details
                    )
                    
            except Exception as e:
                logger.error(f"Consistency check {check.name} failed: {e}")
        
        return results
```

## Key Takeaways

1. **Consistency is a spectrum** - Choose the right consistency model for your use case
2. **Embrace eventual consistency** - In distributed systems, eventual consistency is often most practical
3. **Design for conflicts** - Conflicts are inevitable, design resolution strategies upfront
4. **Use appropriate patterns** - Choose between 2PC, saga, and outbox patterns based on requirements
5. **Monitor consistency** - Set up monitoring to detect and alert on consistency issues

## Next Steps

Continue to [7.5 Scalability Considerations](./system-considerations-75-scalability-considerations.md) to learn about designing systems that can handle growth and increased load.
