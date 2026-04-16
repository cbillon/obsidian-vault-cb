
## A Deep Dive into Building Payment Infrastructure that Processes $100M+ Daily while Maintaining 99.99% Uptime

Picture this: Black Friday 2024, 9:00 AM EST. Transaction volume suddenly spikes to 200,000 requests per minute – 4x normal load. Payment provider response times start creeping up. Risk of payment timeouts increases. This isn't a hypothetical scenario – it's a real stress test that payment systems face during peak seasons.

In this article, I'll share how we built a payment processing system that handles over 1 million daily transactions while maintaining 99.99% uptime. We'll explore the architecture decisions, battle-tested patterns, and hard-learned lessons that make this possible.

## Understanding the Scale

Let's break down what processing a million transactions daily means in practice:

- Each enterprise client processes ~100,000 transactions daily
    
- Every transaction requires 10+ internal API calls
    
- Each transaction must complete in under 500ms
    
- System handles multiple payment methods across 30+ countries
    
- Zero tolerance for double-charges or lost payments
    
- Compliance with PCI-DSS, GDPR, and regional regulations
    

## Core Architecture: The State Machine Foundation

At the heart of our payment system lies a robust state machine. Why? Because payment processing isn't a simple success/failure operation – it's a complex journey through multiple states, each requiring careful handling and perfect audit trails.
![[Pasted image 20260416103847.png]]


Here's how we implement this state machine:

```python
class PaymentState(Enum):
    INITIATED = "initiated"
    PAYMENT_METHOD_VALIDATED = "payment_method_validated"
    FRAUD_CHECKED = "fraud_checked"
    FUNDS_RESERVED = "funds_reserved"
    PROCESSING = "processing"
    COMPLETED = "completed"
    FAILED = "failed"
    REFUND_PENDING = "refund_pending"
    REFUNDED = "refunded"
    DISPUTED = "disputed"
    DISPUTE_RESOLVED = "dispute_resolved"
    EXPIRED = "expired"

class Payment:
    def __init__(self, id: str, amount: float, currency: str):
        self.id = id
        self.amount = amount
        self.currency = currency
        self.state = PaymentState.INITIATED
        self.state_history = [(PaymentState.INITIATED, datetime.utcnow())]

    async def transition(self, new_state: PaymentState) -> bool:
        if not self._is_valid_transition(new_state):
            raise InvalidTransitionError(
                f"Cannot transition from {self.state} to {new_state}"
            )

        self.state_history.append((new_state, datetime.utcnow()))
        self.state = new_state
        return True
```

## Building Resilience: Circuit Breakers and Retries

When dealing with multiple payment providers, failures are inevitable. We implement both circuit breakers and intelligent retry strategies to handle these failures gracefully.
![[Pasted image 20260416104003.png]]
ProviderCircuit BreakerRetry HandlerClientProviderCircuit BreakerRetry HandlerClientCalculate BackoffWait for Backoffalt[Success][Temporary Failure]alt[Circuit Closed][Circuit Open]loop[Until success or max attempts]Process PaymentCheck CircuitProcess PaymentSuccess ResponseSuccess ResponseSuccess ResponseTemporary ErrorRetryable ErrorCircuit Open ErrorProvider Unavailable

Our circuit breaker implementation:

```python
class CircuitBreaker:
    def __init__(
        self,
        failure_threshold: int = 5,
        recovery_timeout: int = 60,
        half_open_max_calls: int = 3
    ):
        self.state = CircuitState.CLOSED
        self.failures = 0
        self.last_failure_time = None
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout

    async def call(self, func: Callable, *args, **kwargs) -> Any:
        if not self._can_execute():
            raise CircuitOpenError()

        try:
            result = await func(*args, **kwargs)
            self._handle_success()
            return result
        except Exception as e:
            self._handle_failure()
            raise e
```

## Payment Method Integration: The Strategy Pattern

Different payment methods have different requirements and processing flows. We use the strategy pattern to handle this complexity:

```python
class PaymentMethodStrategy(ABC):
    @abstractmethod
    async def validate(self, context: PaymentMethodContext) -> bool:
        pass

    @abstractmethod
    async def process(self, context: PaymentMethodContext) -> Dict:
        pass

    @abstractmethod
    async def refund(self, context: PaymentMethodContext) -> Dict:
        pass

class CreditCardStrategy(PaymentMethodStrategy):
    def __init__(self, stripe_client):
        self.stripe = stripe_client

    async def process(self, context: PaymentMethodContext) -> Dict:
        return await self.stripe.create_charge(
            amount=context.amount,
            currency=context.currency,
            card=context.metadata['card_details'],
            idempotency_key=context.idempotency_key
        )
```

## Real-World Performance Insights

After running this system in production for over a year, here are our key metrics:

- Average transaction time: 300ms
    
- 95th percentile: 450ms
    
- Peak throughput: 3,000 TPS
    
- System availability: 99.99%
    
- Failed transaction rate: <0.1%
    

### Monitoring Approach

We implement comprehensive monitoring for every aspect of the system:

```python
class PaymentMetrics:
    def __init__(self):
        self.metrics_client = MetricsClient()

    def record_metrics(
        self,
        provider: str,
        success: bool,
        duration_ms: float,
        state: str
    ):
        self.metrics_client.timing(
            "payment.duration",
            value=duration_ms,
            tags={
                "provider": provider,
                "success": str(success),
                "state": state
            }
        )

        self.metrics_client.increment(
            "payment.attempts",
            tags={
                "provider": provider,
                "success": str(success)
            }
        )
```

## Lessons Learned and Best Practices

1. **Design for Failure**
    
    - Every external call will fail eventually
        
    - Network issues are more common than you think
        
    - Provider downtimes don't follow schedules
        
2. **Implement Proper Backoff**
    
    - Simple retries can make things worse
        
    - Different scenarios need different strategies
        
    - Monitor retry effectiveness
        
3. **Monitor Everything**
    
    - Track all state transitions
        
    - Measure provider performance
        
    - Set up alerts for anomalies
        
4. **Keep It Simple**
    
    - Complex systems fail in complex ways
        
    - Clear patterns are easier to debug
        
    - Simplicity enables reliability
        

## Conclusion: Beyond the Million Transaction Mark

Building a payment system that handles millions of transactions isn't just about writing code – it's about understanding the intricate dance between different systems, anticipating failure modes, and building resilience at every layer.

The combination of state machines, circuit breakers, and smart retry strategies has enabled us to build a system that not only handles the scale but does so reliably and predictably. As we continue to grow and process even more transactions, these patterns and practices continue to serve as the foundation of our system's reliability.

Remember: in payment processing, reliability isn't a feature – it's a requirement. Every failed transaction represents a frustrated customer and potential lost business. By implementing these patterns and learning from real-world scenarios, you can build a payment system that your customers can rely on, even as you scale to millions of transactions and beyond.