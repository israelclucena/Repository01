it('não deve chamar updateIframeHeight se a URL não mudar', () => {
  spyOn(component, 'updateIframeHeight');

  const changes: SimpleChanges = {
    ssoToken: { previousValue: 'old-token', currentValue: 'new-token', firstChange: false, isFirstChange: () => false }
  };

  component.ngOnChanges(changes);

  expect(component.updateIframeHeight).not.toHaveBeenCalled();
});
