# test
using System;
using System.Windows.Input;
using Microsoft.Maui.Controls;

public class EventToCommandBehavior : Behavior<View>
{
    public static readonly BindableProperty EventNameProperty =
        BindableProperty.Create(nameof(EventName), typeof(string), typeof(EventToCommandBehavior));

    public static readonly BindableProperty CommandProperty =
        BindableProperty.Create(nameof(Command), typeof(ICommand), typeof(EventToCommandBehavior));

    public static readonly BindableProperty CommandParameterProperty =
        BindableProperty.Create(nameof(CommandParameter), typeof(object), typeof(EventToCommandBehavior));

    private EventInfo _eventInfo;
    private Delegate _handler;

    public string EventName
    {
        get => (string)GetValue(EventNameProperty);
        set => SetValue(EventNameProperty, value);
    }

    public ICommand Command
    {
        get => (ICommand)GetValue(CommandProperty);
        set => SetValue(CommandProperty, value);
    }

    public object CommandParameter
    {
        get => GetValue(CommandParameterProperty);
        set => SetValue(CommandParameterProperty, value);
    }

    protected override void OnAttachedTo(View bindable)
    {
        base.OnAttachedTo(bindable);
        if (!string.IsNullOrEmpty(EventName))
        {
            _eventInfo = bindable.GetType().GetEvent(EventName);
            if (_eventInfo != null)
            {
                _handler = Delegate.CreateDelegate(_eventInfo.EventHandlerType, this, nameof(OnEventRaised));
                _eventInfo.AddEventHandler(bindable, _handler);
            }
        }
    }

    protected override void OnDetachingFrom(View bindable)
    {
        base.OnDetachingFrom(bindable);
        if (_eventInfo != null && _handler != null)
        {
            _eventInfo.RemoveEventHandler(bindable, _handler);
        }
    }

    private void OnEventRaised(object sender, EventArgs e)
    {
        if (Command != null && Command.CanExecute(CommandParameter))
        {
            Command.Execute(CommandParameter);
        }
    }
}
