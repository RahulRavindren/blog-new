---
title: UX flow orchestration in Android
date: 2017-06-08 22:57:18
tags: 
- android
- design patterns
- orchestration 
---

Few months ago I had taken a Freelance mobile project for a company called  Napses Technologies. The requirment was pushing in two apps on the PlayStore that had similar UI but different UX flows at few places. Well, initially I went with a noob way of solving it with **if condition** by checking the **Build.BUILD_TYPE**

After breaking my head with this, I decide on writing an orchestrator to make decision on launching android UI components (Fragments/Activity) based on logic supplied to it. Well here is a take on orchestration that I thought of. Firstly, orchestration has states like success(),failure(),back() 
hex

{% codeblock [OrchestrationState] [lang:java] %}
	
	public interface OrchestrationState {
	    void success(Bundle bundle);
	    void failure();
	    void back();
	}

{% endcodeblock %}

Over here success() executes the fragment transaction, back() is navigation back in the applicaton. This state is used to compose the orchestration as follows.

{% codeblock [OrchestrationState] [lang:java] %}
	
	interface Orchestration {
	    void execute(Bundle bundle);
	    void getBack();
	}

{% endcodeblock %}

The success() of orchestration state calls the execute() to perform fragment transactions. And getBack() handles backstack changes. Well there are 3 stages to orchestration 

1. First orchestration step
2. Final orchestration step
3. Current orchestration step

First and Final orchestration lets me define start and end UX flows. So each fragment is a participant involved in the orchestration. So first orchestration class is implemented like this 


{% codeblock [FirstOrchestrationStep] [lang:java] %}
	
	public class FirstOrchestrationStep implements Orchestration, OrchestrationState {
	    private OrchestrationParticipant participant;
	    private NavigationHandler navigationHandler;
	    private Orchestration nextOrchestration;
	    private Bundle bundle;

	    public FirstOrchestrationStep(OrchestrationParticipant participant, NavigationHandler navigationHandler) {
	        this.participant = participant;
	        this.navigationHandler = navigationHandler;
	        participant.attach(this);
	    }


	    public void state(Orchestration nextOrchestration) {
	        this.nextOrchestration = nextOrchestration;
	    }

	    @Override
	    public void execute(Bundle bundle) {
	        this.bundle = bundle;
	        BaseFragment fragment = participant.associatedFragment();
	        fragment.setArguments(bundle);
	        navigationHandler.addFragment(fragment);
	    }

	    @Override
	    public void getBack() {
	        BaseFragment fragment = participant.associatedFragment();
	        fragment.setArguments(bundle);
	        navigationHandler.addFragment(fragment);
	    }

	    @Override
	    public void success(Bundle bundle) { nextOrchestration.execute(bundle);}

	    @Override
	    public void failure() {	}

	    @Override
	    public void back() {}
	}

{% endcodeblock %}

{% codeblock [FinalOrchestrationStep] [lang:java] %}
	
	public class FinalOrchestrationStep implements Orchestration {

	    private BaseFragment fragment;
	    private NavigationHandler navigationHandler;

	    public FinalOrchestrationStep(BaseFragment fragment, NavigationHandler navigationHandler) {
	        this.fragment = fragment;
	        this.navigationHandler = navigationHandler;
	    }

	    @Override
	    public void execute(Bundle bundle) {
	        fragment.setArguments(bundle);
	        navigationHandler.popAll();
	        navigationHandler.addFragment(fragment);
	    }

	    @Override
	    public void getBack() {}
	}

{% endcodeblock %}

Current orchestration step is similar to **FirstOrchestrationStep**. These define the fragment transations and navigation handler executes them. Well this helped in achieving UX flows in two differen applications. So now we define orchestration for app X and app Y.

{% codeblock [UXFlowAppX] [lang:java] %}
	public class UxFlowAppX {
		public void start{
			FirstOrchestrationStep firstOrchestrationStep = new FirstOrchestrationStep(new FragmentA(), navigationHandler);
	        OrchestrationStep step2 = new OrchestrationStep(new FramentB(), navigationHandler);
	        OrchestrationStep step3 = new OrchestrationStep(new FragmentD(), navigationHandler);
	        OrchestrationStep step4 = new OrchestrationStep(new FragmentF(), navigationHandler);
	        FinalOrchestrationStep step5 = new FinalOrchestrationStep(new FramgnetH(), navigationHandler);

	        firstOrchestrationStep.state(step2);
	        step2.state(step5,step3);
	        step3.state(step3, step4);
	        step4.state(step4, step5);
	        firstOrchestrationStep.execute(new Bundle());
		}
	}
{% endcodeblock %}

Similarly the other application Y would have a UX flow like

{% codeblock [UXFlowAppY] [lang:java] %}
	public class UxFlowAppY {
		public void start{
			FirstOrchestrationStep firstOrchestrationStep = new FirstOrchestrationStep(new FragmentA(), navigationHandler);
	        OrchestrationStep step2 = new OrchestrationStep(new FramentB(), navigationHandler);
	        FinalOrchestrationStep step3 = new FinalOrchestrationStep(new FramgnetH(), navigationHandler);

	        firstOrchestrationStep.state(step2);
	        step2.step(step2,step3);
	        firstOrchestrationStep.execute(new Bundle());
		}
	}
{% endcodeblock %}

 
Well I learnt a lot myself while working on this project. Hope this is helpful to people who fall into the same problem as mine. And open for positive criticism on this over twitter {% link  @the_codebot https://twitter.com/the_codebot [external] [@the_codebot] %}






        