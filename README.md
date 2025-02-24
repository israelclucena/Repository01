it('should call super.ngOnInit and initialize resizingSelectors', async () => {
  spyOn(component, 'ngOnInit').and.callThrough();
  
  await component.ngOnInit();
  
  expect(component.ngOnInit).toHaveBeenCalled();
  expect(component.resizingSelectors).toBeDefined();
});
