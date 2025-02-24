it('should update iframe height and avoid session expiration on ngOnChanges', () => {
  spyOn(component, 'updateIframeHeight');
  spyOn(component, 'eventsAvoidSessionExpired');

  component.ngOnChanges({});
  
  expect(component.updateIframeHeight).toHaveBeenCalled();
  expect(component.eventsAvoidSessionExpired).toHaveBeenCalled();
});
